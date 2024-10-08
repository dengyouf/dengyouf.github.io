---
layout:     post
title:      NoSQL Database of REDIS
subtitle:   
date:       2022-06-03
author:     Deng You
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - redis
---

# NoSQL Database of REDIS

Redis 是一种开源的内存键值数据结构存储，可用作数据库、缓存或消息代理。提供类似功能的还有memcached，但相比memcached，redis还提供了易扩展、高性能、具备数据持久性等功能。

**redis特性**

- 基于内存实现，因此速度很快
- 多种类型的树结构，可为多场景得业务架构提供高效的数据结构
- 单线程处理非堵塞IO，避免了线程切换和竞争消耗
- 支持持久化，高可用和分布式


## redis编译安装

- 编译软件包

```bash
~]# yum -y install gcc jemalloc-devel
~]# wget http://download.redis.io/releases/redis-5.0.7.tar.gz
~]# tar -xf redis-5.0.7.tar.gz
~]# cd redis-5.0.7
redis-5.0.7]# make PREFIX=/data/apps/redis install
redis-5.0.7]# echo 'export PATH=$PATH:/data/apps/redis/bin' >> /etc/profile.d/redis.sh
redis-5.0.7]# .  /etc/profile.d/redis.sh
```

- 准备配置文件

```bash
redis-5.0.7]# mkdir /data/apps/redis/{etc,logs,data,run}
redis-5.0.7]# cp redis.conf /data/apps/redis/etc/
bind 0.0.0.0
port 6379
daemonize no
supervised no
pidfile /data/apps/redis/run/redis_6379.pid
loglevel notice
logfile "/data/apps/redis/logs/redis_6379.log"
dbfilename dump.rdb
dir /data/apps/redis/data
appendonly no
requirepass 'redisPwd'

redis]# touch /data/apps/redis/logs/redis_6379.log
```

- 优化内核参数

```bash
redis-5.0.7]# vim /etc/sysctl.conf
net.core.somaxconn = 1024
vm.overcommit_memory = 1
redis-5.0.7]# sysctl  -p

redis-5.0.7]# echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled'
redis-5.0.7]# echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.d/rc.local
redis-5.0.7]# chmod +x /etc/rc.d/rc.local

redis-5.0.7]# ulimit -n 65535
```

- 启动服务

```bash
~]# useradd -r -s /sbin/nologin redis
~]# chown  redis.redis /data/apps/redis -R
~]# /data/apps/redis/bin/redis-server  /data/apps/redis/etc/redis.conf
~]# pkill -9 redis

~]# vim /usr/lib/systemd/system/redis.service
[Unit]
Description=Redis persistent key-value database
After=network.target
[Service]
ExecStart=/data/apps/redis/bin/redis-server /data/apps/redis/etc/redis.conf --supervised systemd
ExecStop=/bin/kill -s QUIT $MAINPID
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
[Install]
WantedBy=multi-user.target

~]# systemctl  start redis
```



