# Arena 内存管理

因为如果每个客户端请求都向堆申请内存，那么随着请求越来越多，堆中的内存碎片也会越来越多，从而导致需要 GC 的内存空间越来越多。虽然，ZGC 的性能已经非常优秀了，但是还是让 GC 尽量少工作一些，以避免系统延迟抖动和垃圾收集时的 CPU 消耗。

## Arena 类

作用：直接分配一大块内存，申请内存时，每次只能申请固定大小的内存。申请的内存用位图记录，如果某块内存被申请了，那么该内存对应的 bit 被设置为 true。如果该块内存没有被申请，则设置为 false。

```java
public class Arena {
    private final ByteBuffer buffer;
    private final BitSet bitSet;
    private final int allocSize;
    private final int bufferCount;
    private int maxClearBit;

    public Arena(int mainBufferSize, int allocBufferSize) {
        allocSize = allocBufferSize;

        buffer = ByteBuffer.allocateDirect(mainBufferSize);
        bufferCount = mainBufferSize/ allocBufferSize;

        if(mainBufferSize % allocBufferSize != 0) {
            throw new RuntimeException();
        }

        bitSet = new BitSet(bufferCount);
        maxClearBit = 0;
    }

    public synchronized ArenaBuffer alloc() throws ArenaOverflowException {
        // 因为 alloc 内存并不是频繁发生，所以直接使用 cardinality() 计算
        if(bitSet.cardinality() == bufferCount) {
            throw new ArenaOverflowException();
        }

        int nextClearBit = bitSet.nextClearBit(maxClearBit);
        bitSet.set(nextClearBit);
        maxClearBit = (nextClearBit + 1) % bufferCount;

        ByteBuffer allocBuffer = buffer.slice(nextClearBit * allocSize, allocSize);
        return new ArenaBuffer(nextClearBit, allocBuffer);
    }

    public synchronized void dealloc(ArenaBuffer arenaBuffer) {
        int bit = arenaBuffer.bit();
        bitSet.clear(bit);
    }
}
```

## ArenaBuffer 类

作用：ArenaBuffer 是 Arena 分配的固定大小的内存。ArenaBuffer 可以再次被分配成不固定大小的 Segment 块。

```java
public class ArenaBuffer {
    private final int bit;
    private final ByteBuffer buffer;
    private final BitSet bitSet;

    public ArenaBuffer(int bit, ByteBuffer buffer) {
        this.bit = bit;
        this.buffer = buffer;
        this.bitSet = new BitSet(buffer.limit());
    }

    public int bit() {
        return bit;
    }

    public int position() {
        return buffer.position();
    }

    public synchronized void read(SocketChannel channel) throws IOException {
        int oldPosition = buffer.position();
        channel.read(buffer);
        int newPosition = buffer.position();

        if(oldPosition == newPosition) {
            return;
        }

        bitSet.set(oldPosition, newPosition, true);
    }

    public synchronized boolean notFull() {
        return BufferUtils.isNotOver(buffer);
    }

    public synchronized boolean isClear() {
        return bitSet.isEmpty();
    }

    public synchronized Segment alloc(int offset, int length) {
        bitSet.set(offset, offset + length, true);
        ByteBuffer segmentBuffer = buffer.slice(offset, length).asReadOnlyBuffer();
        return new Segment(this, offset, length, segmentBuffer);
    }

    public synchronized void dealloc(Segment segment) {
        int offset = segment.offset();
        int length = segment.length();
        bitSet.clear(offset, offset + length);
    }
}
```

## Segment 类

作用：Segment 是只读的 Buffer，主要是由业务逻辑使用。

```java
public record Segment(
        ArenaBuffer parent,
        int offset,
        int length,
        ByteBuffer buffer
) {
    public static void deallocAll(Segment[] segments) {
        Arrays.stream(segments).forEach(Segment::dealloc);
    }

    public static Buffers buffers(Segment[] segments) {
        ByteBuffer[] buffers = Arrays.stream(segments)
                .map(Segment::buffer)
                .toArray(ByteBuffer[]::new);
        return new Buffers(buffers);
    }

    public void dealloc() {
        parent.dealloc(this);
    }
}
```

## Arena 在 LynxDB Socket 中的使用

**ArenaAllocator**

作用：ArenaAllocator 是一个单例的 Arena 内存分配器。

```java
public class ArenaAllocator {
    public static final int ARENA_BUFFER_SIZE = 1024 * 8;
    private static final Arena arena = new Arena(1024 * 1024 * 1024, ARENA_BUFFER_SIZE);
    public static ArenaBuffer alloc() {
        try {
            return arena.alloc();
        } catch (ArenaOverflowException ignored) {
            // TODO 处理 Arena 溢出问题
            throw new RuntimeException();
        }
    }

    public static void dealloc(ArenaBuffer arenaBuffer) {
        arena.dealloc(arenaBuffer);
    }
}
```

**ArenaBufferManager**

作用：管理从 Arena 中分配的内存，并按照一定的规则生成 Segement。

```java
public class ArenaBufferManager {
    private final List<ArenaBuffer> arenaBuffers = new LinkedList<>();
    private volatile int position = 0;

    public ArenaBuffer readableArenaBuffer() {
        if(arenaBuffers.isEmpty()) {
            ArenaBuffer newArenaBuffer = ArenaAllocator.alloc();
            arenaBuffers.addLast(newArenaBuffer);
            return newArenaBuffer;
        }

        ArenaBuffer arenaBuffer = arenaBuffers.getLast();
        if(arenaBuffer.notFull()) {
            return arenaBuffer;
        }

        ArenaBuffer newArenaBuffer = ArenaAllocator.alloc();
        arenaBuffers.addLast(newArenaBuffer);
        return newArenaBuffer;
    }

    public void dealloc() {
        arenaBuffers.forEach(ArenaAllocator::dealloc);
    }

    public boolean notEnoughToRead(int length) {
        if(arenaBuffers.isEmpty()) {
            throw new RuntimeException();
        }

        int size = arenaBuffers.size();
        int total = arenaBuffers.getLast().position()
                + (size - 1) * ArenaAllocator.ARENA_BUFFER_SIZE;

        return position + length > total;
    }

    public Segment[] read(int length, boolean isPositionChange) {
        // 检查数据足够吗
        if(notEnoughToRead(length)) {
            throw new RuntimeException();
        }

        int idx = position / ArenaAllocator.ARENA_BUFFER_SIZE;
        int size = arenaBuffers.size();

        List<Segment> segments = new ArrayList<>();
        int tempPosition = position, remainingLength = length;
        for(int i = idx; i < size && remainingLength > 0; i ++) {
            ArenaBuffer arenaBuffer = arenaBuffers.get(i);
            int bufferPosition = arenaBuffer.position();

            int readPosition = tempPosition % ArenaAllocator.ARENA_BUFFER_SIZE;
            int readLength = Math.min(remainingLength, bufferPosition - readPosition);

            Segment segment = arenaBuffer.alloc(readPosition, readLength);
            segments.add(segment);

            remainingLength -= readLength;
            tempPosition += readLength;
        }

        if(isPositionChange) {
            position += length;
        }

        return segments.toArray(Segment[]::new);
    }

    public int readInt(boolean isPositionChange) {
        Segment[] segments = read(INT_LENGTH, isPositionChange);
        if(segments.length == 1) {
            int value = segments[0].buffer().getInt();
            // 返还分配的内存
            Segment.deallocAll(segments);
            return value;
        }

        ByteBuffer intBuffer = BufferUtils.intByteBuffer();
        for (Segment segment : segments) {
            intBuffer.put(segment.buffer());
        }
        // 返还分配的内存
        Segment.deallocAll(segments);
        return intBuffer.rewind().getInt();
    }

    public void incrementPosition(int length) {
        position += length;
    }

    public void clearFreeBuffers() {
        ArenaBuffer arenaBuffer;
        while (!arenaBuffers.isEmpty()
                && (arenaBuffer = arenaBuffers.getFirst()).isClear()) {
            ArenaAllocator.dealloc(arenaBuffer);
            arenaBuffers.removeFirst();
            position -= ArenaAllocator.ARENA_BUFFER_SIZE;
        }
    }
}
```
