---
layout:     post
title:      Installing MySQL on Linux Using Generic Binaries
subtitle:   
date:       2022-12-23
author:     Deng You
header-img: img/post-bg-mysql.jpg
catalog: true
tags:
    - mysql
---


# MySQL安装

## 通用二进制安装

### mysql 8.0 安装
> [官网](https://dev.mysql.com/doc/refman/8.4/en/binary-installation.html): `https://dev.mysql.com/doc/refman/8.4/en/binary-installation.html`

- 安装准备

```shell
# 1. 关闭防火墙
~]# systemctl stop firewalld && systemctl disable firewalld
# 2. 关闭selinux
~]# setenforce 0  && sed -i 's@SELINUX=enforcing@SELINUX=disabled@g' /etc/selinux/config
# 3. 时间同步
~]# crontab  -l
*/3 * * * * /usr/sbin/ntpdate  ntp.aliyun.com &> /dev/null
```

- 安装依赖

```shell
# 1. 清理系统自带的软件包
~]# yum remove -y `rpm -qa|grep mariadb`
# 2. 安装依赖
~]# yum install -y libaio-devel

# (centos8可选操作)执行下面的的命令（任意其一）
~]# ln -s /usr/lib64/libncurses.so.6 /usr/lib64/libncurses.so.5
~]# yum install ncurses-compat-libs

# 3. 系统限制
~]# cat >> /etc/security/limits.conf <<EOF
mysql hard nofile 65535
mysql soft nofile 65535
EOF
```

- 获取软件包

```shell
# 1. 查看系统 glibc 版本
~]# getconf GNU_LIBC_VERSION
glibc 2.17
# 2. 下载软件包
~]# wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.39-linux-glibc2.12-x86_64.tar
```

- 解压安装

```shell
# 1. 添加 mysql用户 
~]# useradd  mysql
# 2. 解压软件包
~]# mkdir -pv /data/apps
~]# tar -xf mysql-8.0.39-linux-glibc2.12-x86_64.tar
~]# tar -xf mysql-8.0.39-linux-glibc2.12-x86_64.tar.xz  -C /data/apps/
# 3. 创建软链
~]# ln -sv /data/apps/mysql-8.0.39-linux-glibc2.12-x86_64 /data/apps/mysql
# 4. 创建数据目录并授权
~]# mkdir -pv /data/apps/mysql/{data,logs}
~]# chown -R  mysql. /data/apps/mysql*
```
- 配置环境变量

```shell
cat  >> /etc/profile.d/tools.sh <<EOF
PATH=$PATH:/data/apps/mysql/bin/
export PATH
EOF
source  /etc/profile.d/tools.sh
```

- 非安全方式初始化数据库

`--initialize-insecure`表示非安装方式初始化，用户可以直接登陆没有初始密码
```shell
~]# /data/apps/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/data/apps/mysql --datadir=/data/apps/mysql/data
2024-09-04T07:24:39.582052Z 0 [System] [MY-013169] [Server] /data/apps/mysql/bin/mysqld (mysqld 8.0.39) initializing of server in progress as process 21                           581
2024-09-04T07:24:39.639471Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2024-09-04T07:24:51.243902Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2024-09-04T07:25:07.112434Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --init                           ialize-insecure option.
```

- 提供配置文件`my.cnf`

```shell
cat > /etc/my.cnf <<EOF
[mysql]
socket=/data/apps/mysql/data/mysql.sock
[mysqld]
innodb_fast_shutdown=0
user=mysql
basedir=/data/apps/mysql
datadir=/data/apps/mysql/data
socket=/data/apps/mysql/data/mysql.sock
general_log = 1
general_log_file =  /data/apps/mysql/logs/general.log
log_error = /data/apps/mysql/logs/error.log
log_timestamps=SYSTEM
log-bin = /data/apps/mysql/logs/mysql-bin
server_id=225
default_authentication_plugin=mysql_native_password
EOF
```

- 提供启动脚本

```shell
# --> 方式1
# 1. 提供启动脚本
~]# cp /data/apps/mysql/support-files/mysql.server /etc/init.d/mysqld
# 2. 将service管理服务方式转换为systemctl管理
~]# systemctl  start mysqld
# 3. 开机启动
~]# chkconfig --add mysqld
~]# /sbin/chkconfig mysqld on
# 4. 查看是否启动
~]# netstat -lntup|grep 3306
tcp6       0      0 :::3306                 :::*                    LISTEN      21859/mysqld
tcp6       0      0 :::33060                :::*                    LISTEN      21859/mysqld
```
```shell
# --> 方式2
~]# cat > /etc/systemd/system/mysqld.service <<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart= /data/apps/mysql/bin/mysqld --defaults-file=/etc/my.cnf
LimitNOFILE=5000 
EOF
~]# systemctl  daemon-reload
~]# systemctl  start mysqld
```
> 在启动mysql数据库服务时，可能看到33060端口信息，此端口信息主要实现mysqlx协议的通讯过程；利用mysqlx协议可以实现利用mysql-shell功能组件，对数据进行key-value操作，即识别json文件信息，进行远程管理



```shell
~]# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.39 MySQL Community Server - GPL

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select version();
+-----------+
| version() |
+-----------+
| 8.0.39    |
+-----------+
1 row in set (0.00 sec)
```

- 安全方式初始化数据库

`--initialize`表示安全方式初始化，有随机默认管理员密码。使用安全模式初始化数据库后，需要利用临时密码登录数据库服务，使用临时密码只是能登录数据库，如果要管理数据库需要重置管理员用户密码信息。

```shell
# 1. 停止数据库
~]# systemctl  stop mysqld
# 2. 清理数据目录
~]# rm -rf /data/apps/mysql/data/*
# 3. 重新使用安装方式进行初始化
~]# /data/apps/mysql/bin/mysqld --initialize --user=mysql --basedir=/data/apps/mysql --datadir=/data/apps/mysql/data
2024-09-04T07:48:21.900699Z 0 [System] [MY-013169] [Server] /data/apps/mysql/bin/mysqld (mysqld 8.0.39) initializing of server in progress as process 22171
2024-09-04T07:48:21.951912Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2024-09-04T07:48:33.758118Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2024-09-04T07:48:53.860244Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: d?ud=:wk*3#A
# 4. 启动数据库
~]# systemctl  start mysqld
# 5. 连接数据库， 并修改密码
~]# mysql -uroot -p'd?ud=:wk*3#A'
mysql>  alter user root@'localhost' identified by '123456';
Query OK, 0 rows affected (0.09 sec)
```

- mysqladmin 修改密码

```shell
~]# mysqladmin -uroot -p'123456' password 'rootPwd' -S /data/apps/mysql/data/mysql.sock
```
- 使用sql语句修改密码

```shell
# mysql 8.0
mysql> alter user root@'localhost' identified by 'Pwd123';
mysql> flush privileges;
# mysql 5.7
mysql> update mysql.user set authentication_string=PASSWORD('Pwd123') where user='root' and host='localhost';
mysql> flush privileges;
# mysql 5.6
mysql> set password for 'oldboy'@'localhost'=PASSWORD('oldboy123');
mysql> flush privileges
```

### 8.0 vs 5.新特性

- 创建用户并授权区别

```shell
# mysql 5.7 可以通过一条命令实现用户创建并授权
mysql>  grant all on mydb.* to app@'%' identified by 'app123';
# mysql 8.0 需要先创建用户，然后才能继续进行授权
mysql> create user  app@'%' identified by 'app123';
mysql> grant all on mydb.* to  app@'%';
```

- 密码插件区别

```shell
# 使用早期版本的mysql客户端连接 8.0的数据库会报错，原因就是密码插件不一致导致的
#  mysql 8.0 的密码插件为： caching_sha2_password ， show variables like '%default_authentication_plugin%
#  mysql 5.7 的密码插件为：mysql_native_password
~]# mysql -h192.168.122.86 -uapp -p'app123'
ERROR 2059 (HY000): Authentication plugin 'caching_sha2_password' cannot be loaded: /usr/lib64/mysql/plugin/caching_sha2_password.so: cannot open shared object file: No such file or directory
```

解决数据库服务升级后，用户密码加密插件影响连接建立问题，可以采取下面两种方案之一： 
1. 运维侧：替换原有默认密码加密插件，更换为历史版本加密插件为mysql_native_passwordl
2. 开发侧：替换客户端连接数据库服务端的驱动程序软件，使之兼容新版本加密插件功能
```shell
# 1. 创建用户时指定密码插件
mysql> create user dengyouf@'%' identified with mysql_native_password by 'dengyouf123';

# 2. 修改已经创建用户加密插件信息
mysql> alter user app@'%'  identified with mysql_native_password by 'app123';

# 3. 全局修改，在my.cnf添加如下内容
[mysqld]
default_authentication_plugin=mysql_native_password
```

- 用户的role支持，实现批量授权

```shell
# 1. 创建2个role信息, 
# - app_rw具有读写修改删除等权限， 
# - app_r只有读权限
mysql> create role app_rw, app_r;
# 2. role绑定权限
mysql> grant select,update,insert,delete on *.* to app_rw;
mysql> grant select on *.* to app_r;

# 4. 创建用户
mysql> create user  read_only_user@'%' identified by 'r123456';
mysql> create user  read_write_user@'%' identified by 'rw123456';

# 5. 将不同的role绑定到不同的用户
mysql> grant app_rw to read_write_user@'%';
mysql> grant app_r to read_only_user@'%';

# 6. 激活role信息
## 方式1： 手动激活
mysql> set default role all to read_only_user@'%';
mysql> set default role all to read_write_user@'%';

## 方式2： 自动激活
mysql > set global activate_all_roles_on_login=on;
```





