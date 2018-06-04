---
title: 使用docker-compose 搭建 MongoDB 集群
date: 2018-05-31 18:32:16
tags: mongo docker docker-compose
---

这里主要介绍Replica Set 模式

1. docker-compose.yml

```
version: '1'
services:
  rs1:
    image: mongo:3.2.11
    volumes:
      - /data/mongo/db/ms1:/data/db
      - /data/mongo/conf/ms1:/data/conf
      - /data/mongo/log/ms1:/data/log
    command: mongod --config /data/conf/mongo.conf
    ports:
      - "27017:27017"
  rs2:
    image: mongo:3.2.11
    volumes:
      - /data/mongo/db/ms2:/data/db
      - /data/mongo/conf/ms2:/data/conf
      - /data/mongo/log/ms2:/data/log
    command: mongod --config /data/conf/mongo.conf
    ports:
      - "28017:27017"
  rs3:
    image: mongo:3.2.11
    volumes:
      - /data/mongo/db/ms3:/data/db
      - /data/mongo/conf/ms3:/data/conf
      - /data/mongo/log/ms3:/data/log
    command: mongod --config /data/conf/mongo.conf
    ports:
      - "29017:27017"

```
2. 创建文件夹

```
 mkdir -p /data/mongo/db/ms1 /data/mongo/db/ms2 /data/mongo/db/ms2
 mkdir -p /data/mongo/conf/ms1 /data/mongo/conf/ms2 /data/mongo/conf/ms2
 mkdir -p /data/mongo/log/ms1 /data/mongo/log/ms2 /data/mongo/log/ms2
```
这里的三个路径分别存放数据，配置及日志。

3. mongod.conf

```
# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /data/db
  journal:
    enabled: true

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /data/log/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0

replication:
  replSetName: bitsowit

```

将配置文件 mongod.conf 拷贝至三个节点对应的配置文件目录下

4. 启动MongoDB 集群

```
docker-compose up -d
```
这行命令会将三个几点都启动

5. 集群初始化

登录ms1这个节点
```
docker-compose exec ms1 mongo
```
通过这条命令可以连接到ms1这个节点的客户端

接下来我们进行初始化集群的操作

```
rs.initiate()
rs.add('ms2:27017')
rs.add('ms3:27017')
rs.conf() //查看配置
rs.status() //查看集群状态
```
