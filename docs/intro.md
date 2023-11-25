# LynxDB 数据库（Version 1.0）

LynxDB 支持（列族，列，键，值）的表结构存储，存储引擎基于 LSM Tree 实现。目前，LynxDB 支持插入，查询，删除和范围查找等数据操作，不支持事务。LynxDB 使用 Java 语言编写，目前只有 Java 客户端，其他语言的客户端将会在以后补充。

LynxDB 在 Version 1.0 版本不支持分布式集群，基于 Raft 的高可用集群将会在以后的版本中实现。LynxDB 1.0 的主要目标是实现一个稳定以及高性能的单机数据库，并为以后的数据库集群提供高性能节点。

> **LynxDB 的版本规则**
>
> 开发版本以发布的时间作为版本号，例如 `2023.10.5-snapshot`，正式的版本则会以 `1.0.0` 作为版本号。主要是为了能够很清晰的了解开发版本的发布时间。

## 功能简介

LynxDB 支持插入（`insert`），查询 Key 的值（`find`），删除（`delete`），查询 Key 在主列上是否存在（`exist`），向后的范围查找（`range-next`），向前的范围查找（`range-before`）。

## 项目启动

LynxDB 项目由 Java 语言编写，可以直接用 `java -jar` 的方式启动。其中 `lynxdb-server-*.jar` 是 LynxDB 服务器的包，`lynxdb-cmd-*.jar` 是 LynxDB 客户端的包。

**启动服务器**

```shell
java -Xmx1g -Xms1g -XX:+UseZGC -jar lib/lynxdb-server-2023.10.5-snapshot.jar
```

**启动客户端**

```shell
java -jar lib/lynxdb-cmd-2023.10.5-snapshot.jar
```

## Systemd 配置

将 lynxdb.service 文件复制到 `/etc/systemd/system/`，然后使用 `systemctl daemon-reload` 命令重新加载配置，使用 `systemctl start lynxdb` 变可以启动 LynxDB 服务。

start-server.sh 文件如下：

```shell
#!/bin/bash
java -Dlynxdb.baseDir=/root/lynxdb-v2023.10.5-snapshot/\
     -Xmx256m -Xms256m\
     -XX:+UseZGC\
     -jar /root/lynxdb-v2023.10.5-snapshot/lib/lynxdb-server-2023.10.5-snapshot.jar
```

lynxdb.service 配置文件如下：

```text
[Unit]
Description=LynxDB Server
After=network.target

[Service]
ExecStart=/root/lynxdb-v2023.10.5-snapshot/start-server.sh
Restart=on-failure
Type=simple

[Install]
WantedBy=multi-user.target
Alias=lynxdb.service
```

## 配置文件

默认配置文件：`[lynxdb-path]/config/app.cfg`

```text
host                    = 127.0.0.1
port                    = 7820
dataDir                 = [base]/data/single/base
runningMode             = single
enableFlightRecorder    = true
```

目前 LynxDB 数据库配置的选项比较少，存储引擎的相关参数将会在以后的版本添加。

配置项说明：

| 配置项               | 描述               |
| -------------------- | ------------------ |
| host                 | ip 或域名          |
| port                 | 端口号             |
| dataDir              | 数据目录           |
| runningMode          | 运行模式           |
| enableFlightRecorder | 是否启动飞行记录器 |
