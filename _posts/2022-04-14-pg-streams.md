
---

title: PG的流复制
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:

---


### postgresql流复制

#### 异步流复制

+ 主库创建复制用户

  ```
  postgres=# create role USERNAME login replication encrypted password 'PASSWORD';
  ```

+ 主库配置`pg_hba.conf`访问控制

  ```
  vim $PGDATA/pg_hba.conf
  host	replication		USERNAME	SLAVE		trust
  ```

+ 修改主库配置`postgres.conf`文件

  ```
  vim $PGDATA/postgres.conf
  listen_address = '*' # 设置监听地址，*代表本机所有地址
  wal_level = replica # 设置日志级别，一般设置成replca就行
  max_wal_senders = 10 # 设置wal日志发送最多10并行 
  archive_mode = on # 开启日志归档
  archive_command = 'cp %p /home/postgres/arch_log/%f' # 设置归档命令
  restore_command = 'cp /home/postgres/arch_log/%f %p' # 设置恢复命令
  recovery_target_timeline = 'latest' # 设置恢复到最新的时间线
  ```
  
+ 主库重启数据库

  ```
  [postgres@db01 ~]$ pg_ctl restart -l restart.log
  ```

+ 从库使用`pg_basebackup`拉取主库数据(安装完postgresql后不需要初始化数据库)

  ```
  [postgres@db02 ~]$ pg_basebackup -h MASTER -p PORT -U USERNAME -R -F p -P -D $PGDATA
  -h: 指定主库，使用主机名或者ip地址
  -p: 指定主库端口
  -U: 指定复制用户名
  -R: 创建恢复文件，pg10之前会创建'recovery.conf'文件，10之后改为'standby.signal'文件，所以此参数可省略
  -F p: 指定备份格式为文本格式
  -P: 显示进度
  -D: 指定数据目录
  ```

+ 修改从库配置`postgres.conf`文件

  ```
  vim $PGDATA/postgres.conf
  listen_address = '*' # 设置监听地址，*代表本机所有地址
  wal_level = replica # 设置日志级别，一般设置成replca就行
  max_wal_senders = 10 # 设置wal日志发送最多10并行 
  archive_mode = on # 开启日志归档
  archive_command = 'cp %p /home/postgres/arch_log/%f' # 设置归档命令
  restore_command = 'cp /home/postgres/arch_log/%f %p' # 设置恢复命令
  recovery_target_timeline = 'latest' # 设置恢复到最新的时间线
  #full_page_writes = on # 同步复制配置，这里注释
  #wal_log_hints = on # 同步复制配置，这里注释
  以下参数看需求是否添加:
  hot_standby = on # 在备份的时候允许查询操作，默认开启
  max_standby_streaming_delay = 30s # 可选参数，流复制最大延迟
  wal_receiver_status_interval = 10s # 可选参数，从库向主库报告状态的最大间隔时间
  hot_standby_feedback = on # 可选参数，查询冲突时向主库反馈
  ```
  
+ 从库配置`pg_hba.conf`访问控制

  ```
  vim $PGDATA/pg_hba.conf
  host	replication		USERNAME	MASTER		trust
  ```

+ 从库创建标记文件

  ```
  vim $PGDATA/standby.signal
  primary_conninfo = 'host=MASTER application_name=standby_pg2 port=PORT user=USERNAME password=PASSWORD option="-c wal_senders_timeout=5000"' # 主库连接信息
  restore_command = 'cp /home/postgres/arch_log/%f %p' # 变成主库需要的参数
  archive_cleanup_command = 'pg_archivecleanup /home/postgres/arch_log %r' # 变成主库后需要清空归档日志
  standby_mode = on # 把从库变成read-only transaction模式，不需要进行写操作，允许查询
  ```

+ 启动从库

  ```
  [postgres@db02 ~]$ pg_ctl start -l start.log
  ```

+ 查看各种状态

  - 查看当前是否为从库

    ```
    postgres=# select pg_is_in_recovery();
     pg_is_in_recovery
    -------------------
     f
    (1 row)
    f:表示不处于恢复状态，一般是主库或者不在流复制环境
    t:表示处于恢复状态，一般是从库s
    ```

  - 在主库查询流复制状态

    ```
    postgres=# \x
    Expanded display is on.
    postgres=# select * from pg_stat_replication;
    (0 rows)
    ```

  - 查看控制文件数据确定主从角色

    ```
    [postgres@db01 ~]$ pg_controldata | grep cluster  # 主库
    Database cluster state:		in production
    [postgres@db02 ~]$ pg_controldata | grep cluster  # 从库
    Database cluster state:		in archive recovery 
    ```

+ 实操

  + 环境说明

    ```
    [root@testlab ~]# cat /etc/redhat-release
    CentOS Linux release 7.9.2009 (Core)
    安装好postgresql 13.4
    同一台机器上起两个数据库簇，PGDATA分别是/pg1和/pg2，pg1的监听端口为5432，pg2的监听端口为5433
    ```

  + 创建两个数据目录并赋予权限

    ```
    [root@testlab ~]# mkdir -pv /pg{1,2}
    [root@testlab ~]# chown postgres:postgres /pg{1,2}
    ```

  + 切换到`postgres`用户，初始化`pg1`

    ```
    [postgres@testlab ~]$ initdb -k -D /pg1/
    ```

  + 创建两个数据库的归档日志存放目录

    ```
    [postgres@testlab ~]$ mkdir -pv /home/postgres/pg{1,2}_arch
    ```

  + 启动`pg1`

    ```
    [postgres@testlab ~]$ pg_ctl -D /pg1 -l pg1.log start
    ```

  + 修改`pg1`的配置文件`postgresql.conf`启用wal日志归档，并切换一下日志查看效果

    ```
    vim /pg1/postgresql.conf
    wal_level = replica
    archive_mode = on
    archive_command = 'cp %p /home/postgres/pg1_arch/%f'
    [postgres@testlab ~]$ pg_ctl restart -D /pg1/
    postgres=# select pg_switch_wal();
    [postgres@testlab ~]$ ls pg1_arch/
    000000010000000000000001
    ```

  + `pg1`创建流复制用户

    ```
    postgres=# create role repl login replication encrypted password '123456';
    ```

  + 修改`pg1`的访问控制文件`pg_hba.conf`

    ```
    vim /pg1/pg_hba.conf
    host    all             repl            testlab                 trust
    [postgres@testlab ~]$ psql -p 5432 -U repl -d postgres
    psql (13.4)
    Type "help" for help.
    
    postgres=> \q
    ```

  + 修改`pg1`的配置文件`postgresql.conf`以支持流复制，并重启数据库

    ```
    vim /pg1/postgresql.conf
    restore_command = 'cp /home/postgres/pg1_arch/%f %p'
    recovery_target_timeline = 'latest'
    max_wal_senders = 10
    [postgres@testlab ~]$ pg_ctl restart -D /pg1/ -l pg1.log
    ```

  + 使用`pg_basebackup`从`pg1`复制出`pg2`

    ```
    [postgres@testlab ~]$ pg_basebackup -U repl -R -F p -P -D /pg2/                24288/24288 kB (100%), 1/1 tablespace
    ```

  + 修改`pg2`的配置文件`postgresql.conf`监听端口，归档日志目录

    ```
    vim /pg2/postgresql.conf
    port = 5433
    archive_command = 'cp %p /home/postgres/pg2_arch/%f'
    restore_command = 'cp /home/postgres/pg2_arch/%f %p'
    ```

  + 在`pg2`的数据目录创建流复制标记文件`standby.signal`，并写入主库连接信息和其他配置

    ```
    vim /pg2/standby.signal
    primary_conninfo = 'host=testlab port=5432 user=re
    pl password=123456 option="-c wal_senders_timeout=5000"'
    restore_command = 'cp /home/postgres/pg2_arch/%f %p'
    archive_cleanup_command = 'pg_archivecleanup /home/postgres/pg2_arch %r'
    standby_mode = on
    ```

  + 启动`pg2`并查看是否处于`recovery`状态

    ```
    [postgres@testlab ~]$ pg_ctl -D /pg2 -l pg2.log start
    [postgres@testlab ~]$ psql -p 5433
    psql (13.4)
    Type "help" for help.
    
    postgres=# select pg_is_in_recovery();
     pg_is_in_recovery
    -------------------
     t
    (1 row)
    ```

  + 在`pg1`上查看流复制状态

    ```
    [postgres@testlab ~]$ psql -p 5432
    psql (13.4)
    Type "help" for help.
    
    postgres=# \x
    Expanded display is on.
    postgres=# select * from pg_stat_replication;
    -[ RECORD 1 ]----+------------------------------
    pid              | 1859
    usesysid         | 16384
    usename          | repl
    application_name | walreceiver
    client_addr      |
    client_hostname  |
    client_port      | -1
    backend_start    | 2021-08-27 10:04:40.552649+08
    backend_xmin     |
    state            | streaming
    sent_lsn         | 0/5000148
    write_lsn        | 0/5000148
    flush_lsn        | 0/5000148
    replay_lsn       | 0/5000148
    write_lag        |
    flush_lag        |
    replay_lag       |
    sync_priority    | 0
    sync_state       | async
    reply_time       | 2021-08-27 10:06:00.655356+08
    ```

  + 测试流复制效果

    ```
    [postgres@testlab ~]$ psql -p 5432
    psql (13.4)
    Type "help" for help.
    
    postgres=# create database testdb01;
    CREATE DATABASE
    postgres=# \c testdb01
    You are now connected to database "testdb01" as user "postgres".
    testdb01=# create table t1(id int, name varchar(50));
    CREATE TABLE
    testdb01=# insert into t1 values(1,'a');
    INSERT 0 1
    [postgres@testlab ~]$ psql -p 5433
    psql (13.4)
    Type "help" for help.
    
    postgres=# \l
                                      List of databases
       Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privile
    ges
    -----------+----------+----------+-------------+-------------+-----------------
    ------
     postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
     template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
         +
               |          |          |             |             | postgres=CTc/pos
    tgres
     template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
         +
               |          |          |             |             | postgres=CTc/pos
    tgres
     testdb01  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
    (4 rows)
    
    postgres=# \c testdb01
    You are now connected to database "testdb01" as user "postgres".
    testdb01=# \d
            List of relations
     Schema | Name | Type  |  Owner
    --------+------+-------+----------
     public | t1   | table | postgres
    (1 row)
    
    testdb01=# select * from t1;
     id | name
    ----+------
      1 | a
    (1 row)
    ```

+ 主备倒换

  + 停主库`pg1`

    ```
    [postgres@testlab ~]$ pg_ctl stop -D /pg1/
    waiting for server to shut down.... done
    server stopped
    ```

  + 从库`pg2`提升为主库

    ```
    [postgres@testlab ~]$ pg_ctl -D /pg2/ promote
    waiting for server to promote.... done
    server promoted
    ```

  + 从库`pg2`注释掉`postgresql.auto.conf`中的主库连接字符串

    ```
    vim /pg2/postgresq.auto.conf
    #primary_conninfo = 'user=repl passfile=''/home/postgres/.pgpass'' channel_binding=disable port=5432 sslmode=disable sslcompression=0 ssl_min_protocol_version=TLSv1.2 gssencmode=disable krbsrvname=postgres target_session_attrs=any'
    ```

  + 重启从库`pg2`完成提升主库操作

    ```
    [postgres@testlab ~]$ pg_ctl restart -D /pg2/ -l pg2.log
    waiting for server to shut down.... done
    server stopped
    waiting for server to start.... done
    server started
    [postgres@testlab ~]$ psql -p 5433
    psql (13.4)
    Type "help" for help.
    
    postgres=# select pg_is_in_recovery();
     pg_is_in_recovery
    -------------------
     f
    (1 row)
    ```

  + 主库`pg1`创建`standby.signal`流复制标志文件，并添加连接信息

    ```
    vim /pg1/standby.signal
    primary_conninfo = 'host=testlab port=5433 user=repl password=123456 option="-c
     wal_senders_timeout=5000"'
    restore_command = 'cp /home/postgres/pg1_arch/%f %p'
    archive_cheanup_command = 'pg_archivecleanup /home/postgres/pg1_arch %r'
    standby_mode = on
    ```

  + 在主库`pg1`的配置文件`postgresql.auto.conf`中添加`pg2`连接信息

    ```
    vim /pg1/postgresql.auto.conf
    primary_conninfo = 'user=repl passfile=''/home/postgres/.pgpass'' channel_binding=disable port=5433 sslmode=disable sslcompression=0 ssl_min_protocol_version=TLSv1.2 gssencmode=disable krbsrvname=postgres target_session_attrs=any'
    ```

  + 启动`pg1`并查看是否处于`recovery`状态

    ```
    [postgres@testlab ~]$ pg_ctl -D /pg1 -l pg1.log  start
    waiting for server to start.... done
    server started
    [postgres@testlab ~]$ psql -p 5432
    psql (13.4)
    Type "help" for help.
    
    postgres=# select pg_is_in_recovery();
     pg_is_in_recovery
    -------------------
     t
    (1 row)
    ```

  + 在`pg2`上查看流复制信息

    ```
    [postgres@testlab ~]$ psql -p 5433
    psql (13.4)
    Type "help" for help.
    
    postgres=# \x
    Expanded display is on.
    postgres=# select * from pg_stat_replication;
    (0 rows)
    失败！并没有流复制信息
    ```

  + 查看`pg1`的启动日志，发现报错

    ```
    2021-08-27 10:50:33.421 CST [4214] FATAL:  could not receive data from WAL stream: ERROR:  requested WAL segment 000000020000000000000006 has already been removed
    ```

  + 手工复制`pg2`的归档日志`000000020000000000000006`到`pg1`的归档日志目录下成功看到流复制信息

    ```
    [postgres@testlab ~]$ psql -p 5433
    psql (13.4)
    Type "help" for help.
    
    postgres=# \x
    Expanded display is on.
    postgres=# select * from pg_stat_replication;
    -[ RECORD 1 ]----+------------------------------
    pid              | 4248
    usesysid         | 16384
    usename          | repl
    application_name | walreceiver
    client_addr      |
    client_hostname  |
    client_port      | -1
    backend_start    | 2021-08-27 10:52:52.97345+08
    backend_xmin     |
    state            | streaming
    sent_lsn         | 0/7002228
    write_lsn        | 0/7002228
    flush_lsn        | 0/7002228
    replay_lsn       | 0/7002228
    write_lag        |
    flush_lag        |
    replay_lag       |
    sync_priority    | 0
    sync_state       | async
    reply_time       | 2021-08-27 10:53:23.031747+08
    ```

  + 测试一下倒换之后的流复制效果

    ```
    [postgres@testlab ~]$ psql -p 5433
    psql (13.4)
    Type "help" for help.
    
    postgres=# \l
                                      List of databases
       Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privile
    ges
    -----------+----------+----------+-------------+-------------+-----------------
    ------
     postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
     template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
         +
               |          |          |             |             | postgres=CTc/pos
    tgres
     template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
         +
               |          |          |             |             | postgres=CTc/pos
    tgres
     testdb01  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
    (4 rows)
    
    postgres=# drop database testdb01;
    DROP DATABASE
    postgres=# \q
    [postgres@testlab ~]$ psql -p 5432
    psql (13.4)
    Type "help" for help.
    
    postgres=# \l
                                      List of databases
       Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privile
    ges
    -----------+----------+----------+-------------+-------------+-----------------
    ------
     postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
     template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
         +
               |          |          |             |             | postgres=CTc/pos
    tgres
     template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
         +
               |          |          |             |             | postgres=CTc/pos
    tgres
    (3 rows)
    ```

#### 同步流复制

+ 修改`standby.signal`里的连接字符串

  ```
  primary_conninfo = 'host=MASTER application_name=standby_db02 port=PORT user=USERNAME password=PASSWORD option="-c wal_senders_timeout=5000"'
  ```

+ 在原有的配置文件`postgresql.conf`上增加一行配置

  ```
  synchronous_standby_names = 'standby_db02'
  ```

+ 实操基本等同于异步流复制，这里就不做了

#### 主从切换

+ 关闭主库(db01)

+ 从库(db02)执行命令切换成主库

  ```
  [postgres@db02 ~]$ pg_ctl promote
  ```

+ 从库(db02)注释`postgresql.auto.conf`中的主库连接信息

+ 重启从库(db02)，此时从库切换成主库

+ 主库(db01)创建`standby.signal`标志文件

+ 主库(db01)在`postgresql.auto.conf`中添加新主库(db02)连接信息

+ 启动主库，此时主库(db01)切换成从库

#### 从库使用`pg_rewind`同步主库数据

```
[postgres@db02 ~]$ pg_ctl -m smart stop
[postgres@db02 ~]$ pg_rewind --target_pgdata $PGDATA --source-server='host=MASTER port=PORT user=USERNAME dbname=DBNAME'
```


