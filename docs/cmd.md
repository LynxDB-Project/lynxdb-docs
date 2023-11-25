# 命令行

## Insert 命令

插入数据：

```shell
insert [key] [columnFamily] [column] [value]
```

## Find 命令

查询单个 column：

```shell
find [key] [columnFamily] [column]
```

查询多个 column：

```shell
find [key] [columnFamily]
```

## Delete 命令

删除数据：

```shell
delete [key] [columnFamily] [column]
```

## Exist 命令

查询 Key 是否存在：

```shell
exist [key] [columnFamily] [column]
```

## Range Next 命令

从 beginKey 往后范围查找：

```shell
range-next [columnFamily] [mainColumn] [beginKey] [limit]
```

## Range Before 命令

从 endKey 往前范围查找：

```shell
range-before [columnFamily] [mainColumn] [endKey] [limit]
```
