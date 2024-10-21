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
redis-5.0.7]# mkdir /data/apps/redis/6379/{etc,logs,data,run}
redis-5.0.7]# cp redis.conf /data/apps/redis/6379/etc/
cat  > /data/apps/redis/6379/etc/redis.conf <<EOF
bind 0.0.0.0
port 6379
daemonize no
supervised no
pidfile /data/apps/redis/6379/run/redis_6379.pid
loglevel notice
logfile "/data/apps/redis/6379/logs/redis_6379.log"
dbfilename dump.rdb
dir /data/apps/redis/6379/data
appendonly no
requirepass 'redisPwd'
repl-backlog-size 1mb
repl-backlog-ttl   3600
EOF

redis]# touch /data/apps/redis/6379/logs/redis_6379.log
```

- 优化内核参数

```bash
redis-5.0.7]# vim /etc/sysctl.conf
net.core.somaxconn = 10240
vm.overcommit_memory = 1
redis-5.0.7]# sysctl  -p

redis-5.0.7]# echo never > /sys/kernel/mm/transparent_hugepage/enabled
redis-5.0.7]# echo 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' >> /etc/rc.d/rc.local
redis-5.0.7]# chmod +x /etc/rc.d/rc.local

redis-5.0.7]# ulimit -n 65535
```

- 启动服务

```bash
~]# useradd -r -s /sbin/nologin redis
~]# chown  redis.redis /data/apps/redis -R
~]# /data/apps/redis/bin/redis-server  /data/apps/redis/6379/etc/redis.conf
~]# pkill -9 redis

~]# vim /usr/lib/systemd/system/redis-6379.service
[Unit]
Description=Redis persistent key-value database
After=network.target
[Service]
ExecStart=/data/apps/redis/bin/redis-server /data/apps/redis/6379/etc/redis.conf --supervised systemd
ExecStop=/bin/kill -s QUIT $MAINPID
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
[Install]
WantedBy=multi-user.target

~]# systemctl  start redis-6379
```

## redis主从同步

虽然redis可以实现淡季的数据持久化，但是无论RDB或者AOF也好，都无法解决单点故障问题，即一旦单台redis服务器节点出现问题，就会造成数据丢失，系统不可用，所以为了解决这个单点故障的问题，可以实现redis主从模式实现跨主机备份。

- 主从复制过程

1）从服务器连接主服务器，发送PSYNC命令
2）主服务器接收到PSYNC命令后，开始执行BGSAVE命令生成RDB快照文件并使用缓冲区记录此后执行的所有
写命令
3）主服务器BGSAVE执行完后，向所有从服务器发送RDB快照文件，并在发送期间继续记录被执行的写命令
4）从服务器收到快照文件后丢弃所有旧数据，载入收到的快照至内存
5）主服务器快照发送完毕后,开始向从服务器发送缓冲区中的写命令
6）从服务器完成对快照的载入，开始接收命令请求，并执行来自主服务器缓冲区的写命令
7）后期同步会先发送自己slave_repl_offset位置，只同步新增加的数据，不再全量同步


一旦redis 成为某个 master的slave，该redis 节点会清空当前服务器上的所有数据，然后讲master的数据导入到自己的内存中，但是如果只是中断了同步关系，则不会删除已经通不过的数据。

### 命令行配置

- 命令行方式

1. 启动主从同步
```
~]# redis-cli  -h 192.168.122.87 -p 6379 -a redisPwd
192.168.122.87:6379> CONFIG SET masterauth redisPwd
192.168.122.87:6379> REPLICAOF 192.168.122.86 6379

192.168.122.86:6379> INFO replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.122.87,port=6379,state=online,offset=378,lag=0
master_replid:e2d53915614a424df8ef89a26c7c5dadf0c56f58
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:378
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:378

```

2. 停止主从复制

停止主从复制可以取消主从同步，但是不会删除slave上的数据

```
192.168.122.87:6379> REPLICAOF no one
OK
192.168.122.87:6379> INFO replication
# Replication
role:master
connected_slaves:0
master_replid:b98e42981067face7f4ff427d702f0e82c1add5b
master_replid2:e2d53915614a424df8ef89a26c7c5dadf0c56f58
master_repl_offset:924
second_repl_offset:925
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:924
192.168.122.87:6379>
```



- 配置文件方式

```
[root@node02 ~]# vim /data/apps/redis/6379/etc/redis.conf
replicaof 192.168.122.86 6379
masterauth 'redisPwd'
```

## redis哨兵模式

主从模式解决了数据单点的问题，但是当主节点宕机以后，数据虽然没有丢失，但是整个系统也会变得不可用，需要人工干预将从节点提升为主节点，同时还要通知应用修改配置文件指向新主节点的IP地址和端口，对于很多应用场景这种中断式的故障处理方式是不可接收的， Redis Sentinel(哨兵)模式就是为了解决这类问题的，其可以自动监听节点的变化，当主节点单机后，自动从其它的从节点中选举一个节点为主节点，并通知其他从节点的主节点指向当前这个新的主节点，从而达到应用户无感知的状态下，完成故障修复，实现 redis 的高可用。

### 配置 Redis Sentinel

哨兵的前提是已经实现了一个redis的主从同步的运行环境，本次实现一个一主两从基于哨兵的高可用
redis架构。master 的配置文件中同样需要配置 masterauth 参数。

1. 配置一主两从环境

```
# node1 节点配置
[root@node01 ~]# cat /data/apps/redis/6379/etc/redis.conf
bind 0.0.0.0
port 6379
daemonize no
supervised no
pidfile /data/apps/redis/6379/run/redis_6379.pid
loglevel notice
logfile "/data/apps/redis/6379/logs/redis_6379.log"
dbfilename dump.rdb
dir /data/apps/redis/6379/data
appendonly no
requirepass 'redisPwd'
masterauth 'redisPwd'

# node2 节点配置
[root@node02 ~]# cat /data/apps/redis/6379/etc/redis.conf
bind 0.0.0.0
port 6379
daemonize no
supervised no
pidfile /data/apps/redis/6379/run/redis_6379.pid
loglevel notice
logfile "/data/apps/redis/6379/logs/redis_6379.log"
dbfilename dump.rdb
dir /data/apps/redis/6379/data
appendonly no
requirepass 'redisPwd'
replicaof 192.168.122.86 6379
masterauth 'redisPwd'

# node3 节点配置
[root@node03 ~]# cat /data/apps/redis/6379/etc/redis.conf
bind 0.0.0.0
port 6379
daemonize no
supervised no
pidfile /data/apps/redis/6379/run/redis_6379.pid
loglevel notice
logfile "/data/apps/redis/6379/logs/redis_6379.log"
dbfilename dump.rdb
dir /data/apps/redis/6379/data
appendonly no
requirepass 'redisPwd'
masterauth 'redisPwd'
replicaof 192.168.122.86 6379

# 查看集群状态
[root@node01 ~]# redis-cli -h 192.168.122.86 -p 6379 -a redisPwd INFO replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.122.87,port=6379,state=online,offset=1960,lag=1
slave1:ip=192.168.122.88,port=6379,state=online,offset=1960,lag=1
master_replid:01505d5a339886c32f5e0bf3f7b14df153d55ce5
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1960
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1960
```

2. 配置哨兵

哨兵通常使用 3 个实例，可以部署在一台机器，使用不同的端口，也可以分别部署到不的机器进行分布式部署

- 准备配置文件和目录

```
mkdir -pv /data/apps/redis-sentinel/6379/{etc,run,logs,data}
cat >  /data/apps/redis-sentinel/6379/etc/redis-sentinel.conf <<EOF
port 26379
daemonize yes
pidfile "/data/apps/redis-sentinel/6379/redis-sentinel.pid"
logfile "/data/apps/redis-sentinel/6379/logs/sentinel.log"
dir "/data/apps/redis-sentinel/6379/data"
sentinel deny-scripts-reconfig yes
sentinel monitor mymaster 192.168.122.86 6379 2
sentinel down-after-milliseconds mymaster 3000
sentinel auth-pass mymaster redisPwd
sentinel config-epoch mymaster 0
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes
EOF
```

- 启动Sentinel 

```
/data/apps/redis/bin/redis-sentinel  /data/apps/redis-sentinel/6379/etc/redis-sentinel.conf &
~]# tailf  /data/apps/redis-sentinel/6379/logs/sentinel.log
```

- 查看sential的状态

```
[root@node02 ~]# redis-cli  -h 192.168.122.86 -p 26379
192.168.122.86:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.122.86:6379,slaves=2,sentinels=1
192.168.122.86:26379> exit
[root@node02 ~]# redis-cli  -h 192.168.122.87 -p 26379
192.168.122.87:26379> INFO sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.122.86:6379,slaves=2,sentinels=1
192.168.122.87:26379> exit
[root@node02 ~]# redis-cli  -h 192.168.122.88 -p 26379
192.168.122.88:26379> INFO sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.122.86:6379,slaves=2,sentinels=1
```

### 停止Master 模拟故障转移

```

[root@node01 ~]# redis-cli  -h 192.168.122.86 -p 6379 -a redisPwd
192.168.122.86:6379> shutdown

# master 节点已经切换为88
[root@node01 ~]# redis-cli  -h 192.168.122.86 -p 26379 -a redisPwd
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.122.86:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.122.88:6379,slaves=2,sentinels=3


192.168.122.88:6379> INFO replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.122.87,port=6379,state=online,offset=48054,lag=0
master_replid:7489f06e686619ec4fdfa28d1d9ff7fe52a072c1
master_replid2:c5722bfd2fd21ddd1edd1f7e26640a978773d386
master_repl_offset:48197
second_repl_offset:14789
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:48197


# 原 master 再次上线后，只能作为 slave 节点，不会抢占成为 master 
[root@node01 ~]# systemctl  start redis-6379
[root@node01 ~]# redis-cli  -h 192.168.122.86 -p 6379 -a redisPwd
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.122.86:6379> info replication
# Replication
role:slave
master_host:192.168.122.88
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:64348
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:7489f06e686619ec4fdfa28d1d9ff7fe52a072c1
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:64348
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:63025
repl_backlog_histlen:1324
```

### Sential 运维

- 手动让主节点下线，切换节点

```
# 修改redis 配置文件， 指定优先级，值越小sentinel会优先将之选为新的master,默为值为100
vim /data/apps/redis/6379/etc/redis.conf
replica-priority 10

[root@node01 ~]# redis-cli  -h 192.168.122.86 -p 26379 -a redisPwd
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.122.86:26379> sentinel failover mymaster
OK
192.168.122.86:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.122.87:6379,slaves=2,sentinels=3
```

## redis集群

哨兵模式虽然解决了redis单点故障问题，但是无法解决redis单机写入的瓶颈，redis写入性能受单机内存大小，并发数量和网卡速率影响。于是，再redis3.0的版本正式推出 Redis Cluster集群，有效的解决了redis分布式部署的需求，当单机内存、并发、流量等瓶颈时，可以采用CLuster架构方法达到负载均衡的目的。

### Redis Cluster 特点

1. 所有节点使用 PING 机制互联
2. 集群节点失效，应由集群中超过半数节点检测失效才算失效
3. 客户端不需要代理即可直接连接redis，应用哦那个程序配置需要有全部的redis IP
4. 采用虚拟槽分区，所有的键根据哈希函数映射到0 ~16383整数槽内

### 集群搭建

集群搭建有三种方式：

1. 依照Redis 协议手工搭建，使用cluster meet、cluster addslots、cluster replicate命令。
2. 5.0之前使用由ruby语言编写的redis-trib.rb，在使用前需要安装ruby语言环境
3. .0及其之后redis摒弃了redis-trib.rb，将搭建集群的功能合并到了redis-cli

这里使用 redis-cli 的方式搭建，三主三从的集群。


-  1. 准备目录
```
~]# mkdir -pv /data/apps/redis/{6379,6380}/{data,logs,etc,logs,run}
```
- 2. 准备配置文件
```
# redis-6379 配置
cat > /data/apps/redis/6379/etc/redis.conf <<EOF
bind 0.0.0.0
port 6379
daemonize no
supervised systemd
pidfile "/data/apps/redis/6379/run/redis_6379.pid"
loglevel notice
logfile "/data/apps/redis/6379/logs/redis_6379.log"
dbfilename "dump.rdb"
dir "/data/apps/redis/6379/data"
appendonly no
requirepass "redisPwd"
masterauth "redisPwd"
maxclients 4064

cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-require-full-coverage no
EOF

# redis-6380 配置
cat > /data/apps/redis/6380/etc/redis.conf <<EOF
bind 0.0.0.0
port 6380
daemonize no
supervised systemd
pidfile "/data/apps/redis/6380/run/redis_6380.pid"
loglevel notice
logfile "/data/apps/redis/6380/logs/redis_6380.log"
dbfilename "dump.rdb"
dir "/data/apps/redis/6380/data"
appendonly no
requirepass "redisPwd"
masterauth "redisPwd"
maxclients 4064

cluster-enabled yes
cluster-config-file nodes-6380.conf
cluster-require-full-coverage no
EOF
```
- 3. 准备服务启动脚本

```
# redis6379.service
cat >  /usr/lib/systemd/system/redis6379.service <<EOF
[Unit]
Description=Redis persistent key-value database
After=network.target
[Service]
ExecStart=/data/apps/redis/bin/redis-server /data/apps/redis/6379/etc/redis.conf  --supervised systemd
ExecStop=/bin/kill -s QUIT $MAINPID
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
[Install]
WantedBy=multi-user.target
EOF

# redis6380.service
cat >  /usr/lib/systemd/system/redis6380.service <<EOF
[Unit]
Description=Redis persistent key-value database
After=network.target
[Service]
ExecStart=/data/apps/redis/bin/redis-server /data/apps/redis/6380/etc/redis.conf  --supervised systemd
ExecStop=/bin/kill -s QUIT $MAINPID
Type=notify
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
[Install]
WantedBy=multi-user.target
EOF
```
- 4. 启动服务

```
~]# chown -R  redis.redis /data/apps/redis
~]# systemctl  start redis6379 redis6380
~]# ss -tnl
State      Recv-Q Send-Q                             Local Address:Port                                            Peer Address:Port
LISTEN     0      511                                            *:6379                                                       *:*
LISTEN     0      511                                            *:6380                                                       *:*
LISTEN     0      128                                            *:22                                                         *:*
LISTEN     0      511                                            *:16379                                                      *:*
LISTEN     0      511                                            *:16380                                                      *:*
LISTEN     0      128                                           :::22                                                        :::*
```

- 5. 创建集群主节点

```
~]# redis-cli --cluster  create 192.168.122.86:6379 192.168.122.87:6379 192.168.122.88:6379 -a redisPwd
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 3 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
M: 990ebd6a3562c0d457228f013f48a3f7b89d42cd 192.168.122.86:6379
   slots:[0-5460] (5461 slots) master
M: b982036c48954b730ed233ca333d142daa782319 192.168.122.87:6379
   slots:[5461-10922] (5462 slots) master
M: 5133768b50014fa944f0d4bbe611031ebb5c6082 192.168.122.88:6379
   slots:[10923-16383] (5461 slots) master
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 192.168.122.86:6379)
M: 990ebd6a3562c0d457228f013f48a3f7b89d42cd 192.168.122.86:6379
   slots:[0-5460] (5461 slots) master
M: 5133768b50014fa944f0d4bbe611031ebb5c6082 192.168.122.88:6379
   slots:[10923-16383] (5461 slots) master
M: b982036c48954b730ed233ca333d142daa782319 192.168.122.87:6379
   slots:[5461-10922] (5462 slots) master
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

~]# redis-cli  -c -h 192.168.122.86 -p 6379 -a redisPwd
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.122.86:6379> INFO cluster
# Cluster
cluster_enabled:1

192.168.122.86:6379> CLUSTER INFO
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:3
cluster_size:3
cluster_current_epoch:3
cluster_my_epoch:1
cluster_stats_messages_ping_sent:158
cluster_stats_messages_pong_sent:155
cluster_stats_messages_sent:313
cluster_stats_messages_ping_received:153
cluster_stats_messages_pong_received:158
cluster_stats_messages_meet_received:2
cluster_stats_messages_received:313


192.168.122.86:6379> cluster NODES
5133768b50014fa944f0d4bbe611031ebb5c6082 192.168.122.88:6379@16379 master - 0 1729510348906 3 connected 10923-16383
990ebd6a3562c0d457228f013f48a3f7b89d42cd 192.168.122.86:6379@16379 myself,master - 0 1729510347000 1 connected 0-5460
b982036c48954b730ed233ca333d142daa782319 192.168.122.87:6379@16379 master - 0 1729510348000 2 connected 5461-10922
```

- 6. 添加集群从节点

```
~]#  redis-cli --cluster add-node 192.168.122.87:6380  192.168.122.86:6379 --cluster-slave --cluster-master-id 990ebd6a3562c0d457228f013f48a3f7b89d42cd  -a redisPwd
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Adding node 192.168.122.87:6380 to cluster 192.168.122.86:6379
>>> Performing Cluster Check (using node 192.168.122.86:6379)
M: 990ebd6a3562c0d457228f013f48a3f7b89d42cd 192.168.122.86:6379
   slots:[0-5460] (5461 slots) master
M: 5133768b50014fa944f0d4bbe611031ebb5c6082 192.168.122.88:6379
   slots:[10923-16383] (5461 slots) master
M: b982036c48954b730ed233ca333d142daa782319 192.168.122.87:6379
   slots:[5461-10922] (5462 slots) master
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 192.168.122.87:6380 to make it join the cluster.
Waiting for the cluster to join
.
>>> Configure node as replica of 192.168.122.86:6379.
[OK] New node added correctly.

~]#  redis-cli --cluster add-node 192.168.122.88:6380  192.168.122.87:6379 --cluster-slave --cluster-master-id b982036c48954b730ed233ca333d142daa782319 -a redisPwd
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Adding node 192.168.122.88:6380 to cluster 192.168.122.87:6379
>>> Performing Cluster Check (using node 192.168.122.87:6379)
M: b982036c48954b730ed233ca333d142daa782319 192.168.122.87:6379
   slots:[5461-10922] (5462 slots) master
M: 990ebd6a3562c0d457228f013f48a3f7b89d42cd 192.168.122.86:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 5133768b50014fa944f0d4bbe611031ebb5c6082 192.168.122.88:6379
   slots:[10923-16383] (5461 slots) master
S: a14c6091eec4b626895124853d96894e6218ac6c 192.168.122.87:6380
   slots: (0 slots) slave
   replicates 990ebd6a3562c0d457228f013f48a3f7b89d42cd
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 192.168.122.88:6380 to make it join the cluster.
Waiting for the cluster to join

>>> Configure node as replica of 192.168.122.87:6379.
[OK] New node added correctly.

~]#  redis-cli --cluster add-node 192.168.122.86:6380  192.168.122.88:6379 --cluster-slave --cluster-master-id 5133768b50014fa944f0d4bbe611031ebb5c6082 -a redisPwd
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Adding node 192.168.122.86:6380 to cluster 192.168.122.88:6379
>>> Performing Cluster Check (using node 192.168.122.88:6379)
M: 5133768b50014fa944f0d4bbe611031ebb5c6082 192.168.122.88:6379
   slots:[10923-16383] (5461 slots) master
S: 7a6618537993ea284ead5dad1660c42c847d490a 192.168.122.88:6380
   slots: (0 slots) slave
   replicates b982036c48954b730ed233ca333d142daa782319
M: 990ebd6a3562c0d457228f013f48a3f7b89d42cd 192.168.122.86:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: a14c6091eec4b626895124853d96894e6218ac6c 192.168.122.87:6380
   slots: (0 slots) slave
   replicates 990ebd6a3562c0d457228f013f48a3f7b89d42cd
M: b982036c48954b730ed233ca333d142daa782319 192.168.122.87:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 192.168.122.86:6380 to make it join the cluster.
Waiting for the cluster to join

>>> Configure node as replica of 192.168.122.88:6379.
[OK] New node added correctly.
```

- 7. 验证添加信息

```
192.168.122.87:6379> CLUSTER NODES
990ebd6a3562c0d457228f013f48a3f7b89d42cd 192.168.122.86:6379@16379 master - 0 1729511257385 1 connected 0-5460
5133768b50014fa944f0d4bbe611031ebb5c6082 192.168.122.88:6379@16379 master - 0 1729511253375 3 connected 10923-16383
7a6618537993ea284ead5dad1660c42c847d490a 192.168.122.88:6380@16380 slave b982036c48954b730ed233ca333d142daa782319 0 1729511256000 2 connected
b982036c48954b730ed233ca333d142daa782319 192.168.122.87:6379@16379 myself,master - 0 1729511253000 2 connected 5461-10922
ead441e9b3b558c4c29d889a2ad5219469a21c11 192.168.122.86:6380@16380 slave 5133768b50014fa944f0d4bbe611031ebb5c6082 0 1729511256381 3 connected
a14c6091eec4b626895124853d96894e6218ac6c 192.168.122.87:6380@16380 slave 990ebd6a3562c0d457228f013f48a3f7b89d42cd 0 1729511254377 1 connected

192.168.122.88:6379> CLUSTER INFO
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:3
cluster_my_epoch:3
cluster_stats_messages_ping_sent:1108
cluster_stats_messages_pong_sent:1152
cluster_stats_messages_meet_sent:2
cluster_stats_messages_sent:2262
cluster_stats_messages_ping_received:1149
cluster_stats_messages_pong_received:1110
cluster_stats_messages_meet_received:3
cluster_stats_messages_received:2262
```

### 集群管理

- 集群信息查看

```
~]# redis-cli --cluster check 192.168.122.86:6379  -a redisPwd   --cluster-search-multiple-owners
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.122.86:6379 (990ebd6a...) -> 0 keys | 5461 slots | 1 slaves.
192.168.122.88:6379 (5133768b...) -> 1 keys | 5461 slots | 1 slaves.
192.168.122.87:6379 (b982036c...) -> 1 keys | 5462 slots | 1 slaves.
[OK] 2 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.122.86:6379)
M: 990ebd6a3562c0d457228f013f48a3f7b89d42cd 192.168.122.86:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: ead441e9b3b558c4c29d889a2ad5219469a21c11 192.168.122.86:6380
   slots: (0 slots) slave
   replicates 5133768b50014fa944f0d4bbe611031ebb5c6082
M: 5133768b50014fa944f0d4bbe611031ebb5c6082 192.168.122.88:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: b982036c48954b730ed233ca333d142daa782319 192.168.122.87:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 7a6618537993ea284ead5dad1660c42c847d490a 192.168.122.88:6380
   slots: (0 slots) slave
   replicates b982036c48954b730ed233ca333d142daa782319
S: a14c6091eec4b626895124853d96894e6218ac6c 192.168.122.87:6380
   slots: (0 slots) slave
   replicates 990ebd6a3562c0d457228f013f48a3f7b89d42cd
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Check for multiple slot owners...


 ~]# redis-cli --cluster info 192.168.122.86:6379  -a redisPwd
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
192.168.122.86:6379 (990ebd6a...) -> 0 keys | 5461 slots | 1 slaves.
192.168.122.88:6379 (5133768b...) -> 0 keys | 5461 slots | 1 slaves.
192.168.122.87:6379 (b982036c...) -> 0 keys | 5462 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.
```

- 修复集群

```
~]# redis-cli  --cluster fix 192.168.122.86:6379  -a redisPwd   --cluster-search-multiple-owners
```














