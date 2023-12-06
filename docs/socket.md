# LynxDB Socket

LynxDB 基于 Java NIO 封装了自己的 Socket 框架，主要使用方法：

> 问：为什么不使用 Netty 作为 Socket 框架？
>
> 答：因为有些代码，你自己不写，就永远不知道性能损失在哪里！

## 创建 Socket 服务器

```java
SocketServerConfig config = new SocketServerConfig(7820);
SocketServer server = new SocketServer(config);
// 设置 handler
server.setHandler(new SocketServerHandler() {
    @Override
    public void handleRequest(SegmentSocketRequest request) {
        // 处理客户端请求...
    }
});
```

## 启动服务器

```java
Executor.start(server);
```

## 发送响应

```java
SelctionKey selectionKey;
int serial;
ByteBuffer[] buffers;
// 构建响应对象
WritableSocketResponse response = new WritableSocketResponse(
    selectionKey,
    serial,
    buffers
);
// 发送响应
server.offerInterruptibly(response);
```

## 创建客户端

```java
SocketClient client = new SocketClient();
// 设置 handler
client.setHandler(new SocketClientHandler() {
    @Override
    public void handleConnected(SelectionKey selectionKey) {
        // 处理 connected 事件...
    }

    @Override
    public void handleResponse(SocketResponse response) {
        // 处理服务端的响应...
    }
});

// 连接服务器
client.connect(new ServerNode("127.0.0.1", 7820));
```

## 启动客户端

```java
Executor.start(client);
```

## 发送请求

```java
SelctionKey selectionKey;
int serial;
ByteBuffer[] buffers;
// 构建 Socket 请求对象
ByteBufferSocketRequest request = new ByteBufferSocketRequest(
    selectionKey,
    serial,
    buffers
);

client.offerInterruptibly(request);
```
