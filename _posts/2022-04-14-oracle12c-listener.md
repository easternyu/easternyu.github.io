---
title: Oracle的监听
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---


### Oracle的监听

#### 常见的问题

1. 我的数据库和监听都启动了，为什么我的客户端还是连不上数据库？
2. 为什么我的监听无法注册服务？
3. 为什么我能tnsping数据库，但是无法连上数据库

#### 监听相关的配置文件

$ORACLE_HOME/network/admin目录下的：

1. listener.ora
2. tnsnames.ora
3. sqlnet.ora

#### 监听相关的参数

1. local_listener：使用非1521端口的动态监听，需要设置这个参数
2. remote_listener

#### 相关的工具

1. netca
2. netmgr

#### 客户端连接数据库流程

客户端向监听器发送连接请求->监听器解析请求并将其转发给所请求的数据库服务处理程序->连接成功

当客户端成功连接数据库之后，监听的启动和关闭不会影响已经连接的会话

每个新的连接，就会产生一个对应的服务器进程（专有服务器连接方式）

连接类型为长连接，如果中间有防火墙，可能会被防火墙阻塞

数据库连接故障时常见的故障之一，和监听文件的正确配置与否关系密切

#### 默认监听的配置文件

```
# listener.ora Network Configuration File: /u01/app/oracle/product/12.2.0/dbhome_1/network/admin/listener.ora
# Generated by Oracle configuration tools.

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = db01)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
```

#### 服务注册

在12c中，使用lreg进程负责监听器的注册（Listener Registration）

从12c中引入了lreg后台进程接管了这部分工作减轻了pmon的工作

一般注册时间周期为1分钟左右

可以使用`SQL> alter system register`手工注册加快进度

一个监听可以同时服务多个数据库实例

#### 监听状态

查看监听状态：

`[oracle@db01 ~]$ lsnrctl status`

READY:就绪状态，代表客户端可以连接了。数据库状态为mount和open状态时可连接

BLOCKED：阻塞状态，客户端无法连接。数据库状态为nomount状态

UNKNOWN：静态监听，nomount状态客户端也可以连接，可以启停实例

#### 添加一个非1521端口的动态监听

1. 修改listener.ora文件，添加内容：

   ```
   LISTENER2 =
     (DESCRIPTION_LIST =
       (DESCRIPTION =
         (ADDRESS = (PROTOCOL = TCP)(HOST = db01)(PORT = 1522))
       )
     )
   ```

2. 修改local_listener参数：

   1. 方法一：先修改tnsnames.ora文件，添加内容；再修改local_listener参数

      ```
      tnsnames.ora文件添加一下内容：
      LISTENER_PROD2 =
        (ADDRESS = (PROTOCOL = TCP)(HOST = db01)(PORT = 1522))
      修改local_listener参数：
      SQL> alter system set local_listener=LISTENER_PROD2;
      ```

   2. 方法二：直接修改local_listener参数

      ```
      SQL> alter system set local_listener='(ADDRESS = (PROTOCOL = TCP)(HOST = db01)(PORT = 1522))';
      ```

      

#### 命名方法

1. Easy Connect

   * 涉及配置文件：sqlnet.ora
   * 只支持tcp/ip，不支持ssl
   * 不支持连接时故障转移、源路由、负载均衡

   ```
   [oracle@db01 ~]$ sqlplus sys/Oracle123@10.21.100.250/PROD as sysdba
   ```

2. 本地连接

   * 支持所有连接
   * 支持连接时故障转移、源路由、负载均衡

3. 目录命名

4. 外部命名

#### 监听负载均衡配置

修改客户端tnsnames.ora文件：

```
PROD =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 10.21.100.250)(PORT = 1521))
    (ADDRESS = (PROTOCOL = TCP)(HOST = 10.21.100.250)(PORT = 1522))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = prod)
    )
    (LOAD_BANLANCE = yes)
  )
```

客户端交替连接到10.21.100.250的1521和1522端口，前提是服务器的监听必须要配置好（我这里默认1521端口，添加了一个1522的监听，监听另外一个实例也可以）

#### 监听的故障转移配置

修改客户端tnsnames.ora文件：

```
PROD =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 10.21.100.250)(PORT = 1521))
    (ADDRESS = (PROTOCOL = TCP)(HOST = 10.21.100.250)(PORT = 1522))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = prod)
    )
    (FAILOVER = ON)
  )
```

如果客户端连接10.21.100.250的1521端口失败，就会连接1522端口，前提是服务器的监听必须要配置好（我这里默认1521端口，添加了一个1522的监听，监听另外一个实例也可以）

#### database link

1. 创建db link

   ```
   SQL> create database link dblink22 connect to sys identified by Oracle123 using '10.21.100.250:1522/prod'; --创建用户私有db link
   SQL> create public database link pubdblink22 connect to sys identitied by Oracle123 using '10.21.100.250:1522/prod'; --创建公共dblink
   ```

2. 删除db link

   ```
   SQL> drop database link dblink22;SQL> drop public database link pubdblink22;
   ```

3. db link的简单实用

   ```sql
   SQL> select * from t1@dblink22; --访问私有db link前必须先授予相应的权限
   SQL> select * from t1@pubdblink22; --公共的db link可以直接使用
   ```

4. 相关视图：`DBA_DB_LINKS`

#### 配置静态监听

1. 使用netmgr配置静态监听

   略

2. 修改listener.ora文件配置静态监听

   ```
   listener.ora添加静态监听：
   LISTENER1 = 
     (DESCRIPTION = 
       (ADDRESS = (PROTOCOL = tcp)(HOST = db01)(PORT = 1522))
     )  
   SID_LIST_LISTENER1 = 
     (SID_LIST = 
       (SID_DESC = 
         (GLOBAL_DBNAME = prod)
         (ORACLE_HOME = /u01/app/oracle/product/12.2.0/dbhome_1) 
         (SID_NAME = prod)
       )
     )
   ```

   
