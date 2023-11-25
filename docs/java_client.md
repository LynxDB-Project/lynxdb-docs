# Java 语言客户端

## Maven 依赖

```xml
<dependency>
    <groupId>com.bailizhang.lynxdb</groupId>
    <artifactId>lynxdb-client</artifactId>
    <version>2023.10.5-snapshot</version>
</dependency>
```

**使用案例**

简单的插入和查询数据的。

```java
public class LynxDbClientDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] key = G.I.toBytes("key");
            byte[] value = G.I.toBytes("value");
            connection.insert(key, "columnFamily", "column", value);

            byte[] findValue = connection.find(key, "columnFamily", "column");
            System.out.println(G.I.toString(findValue));

        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Insert 插入数据

```java
public class InsertKeyDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] key = G.I.toBytes("key");
            byte[] value = G.I.toBytes("value");
            connection.insert(key, "columnFamily", "column", value);

        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Insert 插入多列数据

```java
public class InsertMultiColumnsDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);

            byte[] key = G.I.toBytes("key");
            HashMap<String, byte[]> multiColumns = new HashMap<>();

            for(int i = 0; i < 10; i ++) {
                String column = "column" + i;
                byte[] value = G.I.toBytes("value" + i);

                multiColumns.put(column, value);
            }

            connection.insert(key, "columnFamily", multiColumns);

        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Insert 插入 Java 对象

`@LynxDbColumnFamily("insert-object")` 表示数据插入的 Column Family 为 `"insert-object"`，`@LynxDbKey` 用来标记 Key 字段，`@LynxDbColumn` 用来标记 Column 字段，Java Object 与数据存储中的对应关系为：

Java 对象：

`{key="key", column0="value0", column1="value1", column2="value2"}`

数据表中存储：

| key   | column0  | column1  | column2  |
| ----- | -------- | -------- | -------- |
| "key" | "value0" | "value1" | "value2" |

```java
public class InsertObjectDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);

            DemoObject demoObject = new DemoObject();
            demoObject.setKey("key");
            demoObject.setColumn0("value0");
            demoObject.setColumn1("value1");
            demoObject.setColumn2("value2");

            connection.insert(demoObject);

        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }

    @Data
    @LynxDbColumnFamily("demo-object")
    private static class DemoObject {
        @LynxDbKey
        private String key;

        @LynxDbColumn
        private String column0;

        @LynxDbColumn
        private String column1;

        @LynxDbColumn
        private String column2;
    }
}
```

## Insert 插入超时数据

```java
public class InsertTimeoutKeyDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] key = G.I.toBytes("key");
            byte[] value = G.I.toBytes("value");
            long timeout = System.currentTimeMillis() + 30 * 1000; // 数据在 30s 后超时
            connection.insert(key, "columnFamily", "column", timeout, value);

        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Insert 插入多列超时数据

给多个列同时设置同一个超时时间。

```java
public class InsertMultiTimeoutColumnsDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);

            byte[] key = G.I.toBytes("key");
            HashMap<String, byte[]> multiColumns = new HashMap<>();

            for(int i = 0; i < 10; i ++) {
                String column = "column" + i;
                byte[] value = G.I.toBytes("value" + i);

                multiColumns.put(column, value);
            }

            long timeout = System.currentTimeMillis() + 30 * 1000; // 数据在 30s 后超时

            connection.insert(key, "columnFamily", timeout, multiColumns);

        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Insert 插入超时 Java 对象

```java
public class InsertTimeoutObjectDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);

            DemoObject demoObject = new DemoObject();
            demoObject.setKey("key");
            demoObject.setColumn0("value0");
            demoObject.setColumn1("value1");
            demoObject.setColumn2("value2");

            long timeout = System.currentTimeMillis() + 30 * 1000;

            connection.insert(demoObject, timeout);

        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }

    @Data
    @LynxDbColumnFamily("demo-object")
    private static class DemoObject {
        @LynxDbKey
        private String key;

        @LynxDbColumn
        private String column0;

        @LynxDbColumn
        private String column1;

        @LynxDbColumn
        private String column2;
    }
}
```

## Find 查询数据

```java
public class FindByKeyDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] key = G.I.toBytes("key");
            byte[] value = connection.find(key, "columnFamily", "column");

            System.out.println(G.I.toString(value));
        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Find 查询多个列

```java
public class FindMultiColumnsDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] key = G.I.toBytes("key");
            HashMap<String, byte[]> multiColumns  = connection.findMultiColumns(key, "columnFamily");

            multiColumns.forEach((column, value) -> System.out.println(column + ": " + G.I.toString(value)));
        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Find 查找 Java 对象

```java
public class FindByClassDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            DemoObject demoObject = connection.find("key", DemoObject.class);

            System.out.println(demoObject);
        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }

    @Data
    @LynxDbColumnFamily("demo-object")
    public static class DemoObject {
        @LynxDbKey
        private String key;

        @LynxDbColumn
        private String column0;

        @LynxDbColumn
        private String column1;

        @LynxDbColumn
        private String column2;
    }
}
```

## Exist 查询 Key 是否存在

```java
public class ExistKeyDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] key = G.I.toBytes("key");
            boolean isExisted = connection.existKey(key, "columnFamily", "column");

            System.out.println(isExisted);
        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Exist 查询 Key 是否存在（Java Object 方式）

`@LynxDbMainColumn` 用来标记主列，查询的是被标记的列上 Key 是否存在。

```java
public class ExistKeyByObjectDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);

            DemoObject demoObject = new DemoObject();
            demoObject.setKey("key");
            boolean isExisted = connection.existKey(demoObject);

            System.out.println(isExisted);
        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }

    @Data
    @LynxDbColumnFamily("demo-object")
    public static class DemoObject {
        @LynxDbKey
        private String key;

        @LynxDbMainColumn
        @LynxDbColumn
        private String column0;

        @LynxDbColumn
        private String column1;

        @LynxDbColumn
        private String column2;
    }
}
```

## Range Next 向后的范围查找

```java
public class RangeNextDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] beginKey = G.I.toBytes("");
            List<Pair<byte[], HashMap<String, byte[]>>> multiColumns = connection.rangeNext(
                    "columnFamily",
                    "column",
                    beginKey,
                    10
            );

            multiColumns.forEach(pair -> {
                System.out.println("Key: " + G.I.toString(pair.left()));
                pair.right().forEach((column, value) ->
                        System.out.println("Column: " + column + ", " + "Value: " + G.I.toString(value)));
            });
        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Range Next 向后的范围查找（Java Object 的方式返回）

```java
public class RangeNextObjectDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] beginKey = G.I.toBytes("");
            List<DemoObject> demoObjects = connection.rangeNext(DemoObject.class, beginKey, 10);

            System.out.println(demoObjects);
        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }

    @Data
    @LynxDbColumnFamily("demo-object")
    public static class DemoObject {
        @LynxDbKey
        private String key;

        @LynxDbMainColumn
        @LynxDbColumn
        private String column0;

        @LynxDbColumn
        private String column1;

        @LynxDbColumn
        private String column2;
    }
}
```

## Range Before 向前的范围查找

```java
public class RangeBeforeDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] beginKey = G.I.toBytes("key9");
            List<Pair<byte[], HashMap<String, byte[]>>> multiColumns = connection.rangeBefore(
                    "columnFamily",
                    "column",
                    beginKey,
                    10
            );

            multiColumns.forEach(pair -> {
                System.out.println("Key: " + G.I.toString(pair.left()));
                pair.right().forEach((column, value) ->
                        System.out.println("Column: " + column + ", " + "Value: " + G.I.toString(value)));
            });
        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Range Before 向前的范围查找（Java Object 的方式返回）

```java
public class RangeBeforeObjectDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] beginKey = G.I.toBytes("key9");
            List<DemoObject> demoObjects = connection.rangeBefore(DemoObject.class, beginKey, 10);

            System.out.println(demoObjects);
        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }

    @Data
    @LynxDbColumnFamily("demo-object")
    public static class DemoObject {
        @LynxDbKey
        private String key;

        @LynxDbMainColumn
        @LynxDbColumn
        private String column0;

        @LynxDbColumn
        private String column1;

        @LynxDbColumn
        private String column2;
    }
}
```

## Delete 删除 Key

```java
public class DeleteKeyDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] key = G.I.toBytes("key");
            connection.delete(key, "columnFamily", "column");

        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Delete 删除多个 Column 的 Key

_`deleteMultiColumns()` 方法设计的冗余了，未来将会删掉。_

```java
public class DeleteMultiColumnsDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);
            byte[] key = G.I.toBytes("key");
            connection.deleteMultiColumns(key, "columnFamily", "column", "column1");

        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }
}
```

## Delete 删除 Key（通过 Java Object）

```java
public class DeleteByObjectDemo {
    public static void main(String[] args) {
        G.I.converter(new Converter(StandardCharsets.UTF_8));
        try(LynxDbClient client = new LynxDbClient()) {
            client.start();

            LynxDbConnection connection = client.createConnection("127.0.0.1", 7820);

            DemoObject demoObject = new DemoObject();
            demoObject.setKey("key");

            connection.delete(demoObject);

        } catch (ConnectException e) {
            e.getStackTrace();
        }
    }

    @Data
    @LynxDbColumnFamily("demo-object")
    public static class DemoObject {
        @LynxDbKey
        private String key;

        @LynxDbColumn
        private String column0;

        @LynxDbColumn
        private String column1;

        @LynxDbColumn
        private String column2;
    }
}
```
