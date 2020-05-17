---
title: "Redis使用指南"
date: 2020-05-17 11:00:00 +0800
category: Redis
tags: [Redis, 使用指南]
excerpt: 本文主要主要说明docker下Redis的安装方法及产品特性。
---

# Redis 使用指南

>   REmote DIctionary Server(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统。
> Redis是一个开源的使用ANSIC语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。它通常被称为数据结构服务器，因为值（value）可以是字符串(String), 哈希(Hash), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。

## docker下安装Redis

**1.获取redis镜像**

```shell
$ docker pull redis
```

**2.查看本地镜像** 

```bash
$ docker images
```

**3.设置本地配置文件**

  ①从官网下载[redis.conf](http://download.redis.io/redis-stable/redis.conf)文件

  ②创建本地文件夹，新建配置文件redis.conf贴入从官网下载的配置文件并修改

```bash
$ mkdir /usr/local/docker/redis
$ vi /usr/local/docker/redis/redis.conf
```

  ③修改启动默认配置(从上至下依次)：

- bind 127.0.0.1 #注释掉这部分，这是限制redis只能本地访问


- protected-mode no #默认yes，开启保护模式，限制为本地访问

- daemonize no#默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，改为yes会使配置文件方式启动redis失败

- databases 16 #数据库个数（可选）

- dir  ./ #输入本地redis数据库存放文件夹（可选）

- appendonly yes #redis持久化（可选）

**4.生成redis启动文件**

```bash
$ cd /usr/local/docker/redis
$ cat <<EOF > start.sh
#!/bin/bash
HOST_NAME=localhost
REDIS_DIR=`pwd`
docker stop myredis
docker rm myredis
docker run -d \\
    --hostname \${HOST_NAME} \\
    -p 6379:6379 \\
    --name myredis \\
    -v \${REDIS_DIR}/redis.conf:/etc/redis/redis.conf \\
    -v \${REDIS_DIR}/data:/data \\
    redis redis-server /etc/redis/redis.conf \\
    --appendonly yes
EOF
```

**参数说明**

- `-d`：表示后台启动

- `--hostname \${HOST_NAME}`: 设置访问的域名地址，`${HOST_NAME}`是上面定义的`localhost`这个地址
- `-p 6379:6379`: 把容器内的`6379`端口映射到宿主机`6379`端口
- `-v \${REDIS_DIR}/redis.conf:/etc/redis/redis.conf`：把宿主机配置好的`redis.conf`放到容器内的这个位置中
- `-v \${REDIS_DIR}/data:/data`：把redis持久化的数据在宿主机内显示，做数据备份
- `redis`: `redis`镜像，tag默认版本号是`latest`
- `redis-server /etc/redis/redis.conf`：这个是关键配置，让redis不是无配置启动，而是按照这个`redis.conf`的配置启动
- `–appendonly yes`：redis启动后数据持久化

**5.启动redis**

```bash
$ sh start.sh 
```

**6.查看是否启动成功**

```bash
$ docker ps
```

