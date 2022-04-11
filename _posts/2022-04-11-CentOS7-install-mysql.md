---
title: CentOS7二进制安装MySQL
date: 2022-04-11 00:00:00
categories:
- learn/learn
tags:
---




### CentOS7二进制安装MySQL

#### 1.下载并解压二进制包

```
[root@testlab ~]# wget -O /usr/local/src/mysql.tar.xz http://mirrors.ustc.edu.cn/mysql-ftp/Downloads/MySQL-8.0/mysql-8.0.26-linux-glibc2.12-x86_64.tar.xz
[root@testlab ~]# mkdir -pv /usr/local/src/
[root@testlab ~]# tar -Jxvf /usr/local/src/mysql.tar.xz -C /usr/local/src/mysql --strip-components 1
[root@testlab ~]# mv /usr/local/src/mysql /usr/local/mysql
```

#### 2.创建用户

```
[root@testlab ~]# groupadd mysql
[root@testlab ~]# useradd -r -g mysql -s /bin/false mysql
```

#### 3.移除旧的配置文件

```
[root@testlab ~]# mv /etc/my.cnf /etc/my.cnf.d/ -t /root/mysql-bak/
```

#### 4.创建必要的目录并赋予相应的权限

```
[root@testlab ~]# mkdir -pv /usr/local/mysql/mysql-files
[root@testlab ~]# mkdir -pv /data
[root@testlab ~]# mkdir -pv /var/log/mysql
[root@testlab ~]# chown -R mysql. /usr/local/mysql/mysql-files/
[root@testlab ~]# chmod -R 750 /usr/local/mysql/mysql-files/
[root@testlab ~]# chown -R mysql. /data/
[root@testlab ~]# chmod -R 750 /data/
[root@testlab ~]# chown -R mysql. /var/log/mysql/
```

#### 5.创建配置文件并赋予相应的权限

```
[root@testlab ~]# cat << EOF >> /etc/my.cnf
[mysqld]
datadir=/data
socket=/tmp/mysql.sock
port=3306
log-error=/var/log/mysql/localhost.err
user=mysql
EOF
[root@testlab ~]# chown -R root. /etc/my.cnf
[root@testlab ~]# chmod 644 /etc/my.cnf
```

#### 6.初始化MySQL数据库

```
[root@testlab ~]# /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf --initialize
```

#### 7.从日志中找到初始密码

```
[root@testlab ~]# cat /var/log/mysql/localhost.err  | grep password
2021-08-05T07:17:09.692644Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: kE+mbk<7PWGy
```

#### 8.编写systemd服务文件

```
[root@testlab ~]# cat << EOF >> /usr/lib/systemd/system/mysqld.service
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
Type=notify
TimeoutSec=0
ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf $MYSQLD_OPTS
LimitNOFILE=10000
Restart=on-failure
RestartPreventExitStatus=1
Environment=MYSQLD_PARENT_PID=1
PrivateTmp=false
EOF
```

#### 9.重载systemd的daemon信息，并启动MySQL

```
[root@testlab ~]# systemctl daemon-reload
[root@testlab ~]# systemctl start mysqld
```

#### 10.修改随机生成的密码并查看版本

```
[root@testlab ~]# /usr/local/mysql/bin/mysql -uroot -p'kE+mbk<7PWGy'
mysql> alter user 'root'@'localhost' identified by '123456';
mysql> select version();
+-----------+
| version() |
+-----------+
| 8.0.26    |
+-----------+
```


