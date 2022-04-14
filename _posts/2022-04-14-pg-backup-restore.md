---
title: PG的备份恢复
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---


### postgresql的备份恢复

#### 启用数据库wal日志归档

```
postgres.conf添加配置：
archive_mode = on  # 启用归档模式
archive_command = 'cp %p /home/postgres/arch_log/%f'  # 归档命令
wal_level = replica  # 数据库wal日志级别
然后重启数据库
```

#### 根据toc列表文件恢复指定的表

+ 查看原库中有几张表

  ```
  testdb01=# \d
          List of relations
   Schema | Name | Type  |  Owner
  --------+------+-------+----------
   public | t1   | table | postgres
   public | t2   | table | postgres
  (2 rows)
  ```

+ 完整备份数据库`testdb01`

  ```
  [postgres@db01 ~]$ pg_dump -F t testdb01 > /home/postgres/dbbak/testdb01.tar
  -F t:指定备份成tar包格式
  ```

+ 根据备份文件生成toc列表文件

  ```
  [postgres@db01 ~]$ pg_restore -l /home/postgres/dbbak/testdb01.tar > /home/postgres/dbbak/testdb01.toc
  ```

+ 编辑toc列表文件，注释掉`t2`表的创建和插入语句

  ```
  [postgres@db01 ~]$ vim /home/postgres/dbbak/testdb01.toc
  200; 1259 16437 TABLE public t1 postgres
  ;201; 1259 16440 TABLE public t2 postgres # 用';'注释这条建表语句
  3120; 0 16437 TABLE DATA public t1 postgres
  ;3121; 0 16440 TABLE DATA public t2 postgres # 用';'注释这条插入数据语句
  ```

+ 删除数据库

  ```
  postgres=# drop database testdb01;
  ```

+ 使用`pg_restore`命令恢复数据库

  ```
  [postgres@db01 ~]$ pg_restore -F t -L /home/postgres/dbbak/testdb01.toc --dbname=postgres --create --verbose /home/postgres/dbbak/testdb01.tar
  -F t:表示使用tar包格式的备份文件
  -L:指定列表文件
  --dbname:恢复时连接到哪个数据库，如果事先创建好恢复的目标数据库则连接到目标数据库
  --create:恢复时创建目标数据库，如果事先创建好则不需要此参数
  --verbose:显示恢复过程
  ```

+ 连接`testdb01`数据库查看

  ```
  [postgres@db01 ~]$ psql testdb01
  psql (13.3)
  Type "help" for help.
  
  testdb01=# \d
          List of relations
   Schema | Name | Type  |  Owner
  --------+------+-------+----------
   public | t1   | table | postgres
  ```


#### pg_dumpall备份恢复所有数据库

+ 查看所有数据库

  ```
  postgres=# \l
                                    List of databases
     Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileg
  es
  -----------+----------+----------+-------------+-------------+------------------
  -----
   postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
   template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
      +
             |          |          |             |             | postgres=CTc/post
  gres
   template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
      +
             |          |          |             |             | postgres=CTc/post
  gres
   testdb01  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
   testdb02  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
   testdb1   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
  (6 rows)
  ```

+ 备份所有数据库

  ```
  [postgres@db01 ~]$ pg_dumpall > /home/postgres/dbbak/allbak.sql
  ```

+ 删除几个数据库

  ```
  postgres=# drop database testdb01;
  DROP DATABASE
  postgres=# drop database testdb02;
  DROP DATABASE
  ```

+ 恢复删除的数据库

  ```
  [postgres@db01 ~]$ psql < /home/postgres/dbbak/allbak.sql
  ```

+ 连接数据库查看恢复情况

  ```
  postgres=# \l
                                    List of databases
     Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileg
  es
  -----------+----------+----------+-------------+-------------+------------------
  -----
   postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
   template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
      +
             |          |          |             |             | postgres=CTc/post
  gres
   template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres
      +
             |          |          |             |             | postgres=CTc/post
  gres
   testdb01  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
   testdb02  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
   testdb1   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
  (6 rows)
  postgres=# \c testdb01
  You are now connected to database "testdb01" as user "postgres".
  testdb01=# \d
          List of relations
   Schema | Name | Type  |  Owner
  --------+------+-------+----------
   public | t1   | table | postgres
   public | t2   | table | postgres
  (2 rows)
  testdb01=# select count(*) from t1;
   count
  -------
      47
  (1 row)
  ```


#### 使用`copy`导入导出数据

+ 使用`copy`导出数据

  ```
  testdb01=# copy t1 to '/home/postgres/dbbak/data.txt';
  ```

+ 使用`copy`导入数据

  ```
  testdb01=# create table t3 (id int, name varchar(50));
  CREATE TABLE
  testdb01=# copy t3 from '/home/postgres/dbbak/data.txt';
  COPY 8
  testdb01=# select * from t3;
   id | name
  ----+------
    1 | a
    1 | a
    1 | a
    1 | a
    1 | a
    1 | a
    1 | a
    1 | a
  (8 rows)
  ```

#### 文件系统级别冷备

+ 备份

  ```
  [postgres@db01 ~]$ pg_ctl -m smart stop
  [postgres@db01 ~]$ tar -zcvf /home/postgres/dbbak/allbak.tar.gz $PGDATA
  ```

+ 删除数据目录所有文件

  ```
  [postgres@db01 ~]$ rm -rf $PGDATA/*
  ```

+ 启动数据库报错

  ```
  [postgres@db01 ~]$ pg_ctl start
  pg_ctl: directory "/usr/local/pg133/pgdata" is not a database cluster directory
  ```

+ 恢复数据并启动数据库

  ```
  [postgres@db01 ~]$ tar -zxvf /home/postgres/dbbak/allbak.tar.gz -C /
  [postgres@db01 ~]$ pg_ctl start
  ```

#### 文件系统级别热备

+ 备份(需要数据库处于归档模式下)

  ```
  postgres=# select pg_start_backup('baseline');
  [postgres@db01 ~]$ tar -zcvf /home/postgres/dbbak/hotbak.tar.gz $PGDATA
  postgres=# select pg_stop_backup();
  ```

+ 恢复同冷备

#### 使用`pg_basebackup`备份恢复

+ 备份(需要数据库处于归档模式下)

  ```
  [postgres@db01 ~]$ pg_basebackup -D hotbak1 -Ft -P
  -D:指定数据目录使用环境变量$PGDATA
  -Ft:指定备份格式为tar包
  -P:显示备份过程
  ```

+ 关闭数据库并删除数据目录下所有文件

  ```
  [postgres@db01 ~]$ pg_ctl -m smart stop
  [postgres@db01 ~]$ rm -rf $PGDATA/*
  ```

+ 启动数据库报错

  ```
  [postgres@db01 ~]$ pg_ctl start
  pg_ctl: directory "/usr/local/pg133/pgdata" is not a database cluster directory
  ```

+ 恢复

  ```
  [postgres@db01 ~]$ tar -xvf hotbak1/base.tar -C $PGDATA
  [postgres@db01 ~]$ touch /usr/local/pg133/pgdata/recovery.signal
  还需添加postgres.conf的配置：
  restore_command = 'cp /home/postgres/arch_log/%f %p'
  recovery_target_timeline = 'latest'
  ```

#### 基于lvm快照的备份恢复

略

#### 基于存储点的不完全恢复

+ 创建存储点

  ```
  postgres=# selet pg_create_restore_point('first_pt');
  ```

+ 修改`postgres.conf`配置

  ```
  设置restore_command和recovery_target_name参数：
  resrore_command = 'cp /home/postgres/arch_log/%f %p';
  recovery_target_name = 'first_pt';
  ```

+ 在数据目录创建标记文件

  ```
  [postgres@db01 ~]$ touch $PGDATA/recovery.signal
  ```

+ 正常启动数据库恢复数据

  ```
  [postgres@db01 ~]$ pg_ctl start -l start.log
  ```

+ 执行存储过程，启动新的timeline，使数据库处于可读可写状态

  ```
  postgres=# select pg_wal_replay_resume();
  ```

+ 实操

  - 查看`testdb01`测试库原有信息

    ```
    testdb01=# \d
            List of relations
     Schema | Name | Type  |  Owner
    --------+------+-------+----------
     public | t1   | table | postgres
     public | t2   | table | postgres
    (2 rows)
    
    testdb01=# select count(*) from t1;
     count
    -------
         8
    (1 row)
    
    testdb01=# select count(*) from t2;
     count
    -------
         0
    (1 row)
    ```

  - 做一个基础的全备

    ```
    [postgres@db01 ~]$ pg_basebackup -D hotbak2 -Ft -P
    48134/48134 kB (100%), 1/1 tablespace
    ```

  - 做一些数据修改

    ```
    testdb01=# insert into t2 values(2,'b');
    ```

  - 创建存储点

    ```
    testdb01=# \c postgres postgres
    You are now connected to database "postgres" as user "postgres".
    postgres=# select pg_create_restore_point('first_pt');
     pg_create_restore_point
    -------------------------
     0/D000278
    (1 row)
    ```

  - 做一些操作--比如添加表和删除表

    ```
    testdb01=# create table t3(id int, name varchar(50));
    CREATE TABLE
    testdb01=# drop table t1;
    DROP TABLE
    testdb01=# \d
            List of relations
     Schema | Name | Type  |  Owner
    --------+------+-------+----------
     public | t2   | table | postgres
     public | t3   | table | postgres
    (2 rows)
    ```

  - 恢复到存储点

    ```
    [postgres@db01 ~]$ pg_ctl -m smart stop  # 停数据库
    waiting for server to shut down.... done
    server stopped
    [postgres@db01 ~]$ cd $PGDATA
    [postgres@db01 pgdata]$ rm -rf *  # 删除所有数据
    [postgres@db01 ~]$ tar -xvf hotbak2/base.tar -C $PGDATA  # 恢复全备数据
    [postgres@db01 ~]$ touch $PGDATA/recovery.signal  # 创建恢复标志文件
    [postgres@db01 ~]$ vim $PGDATA/postgresql.conf # 修改配置文件
    recovery_target_name = 'first_pt' # 添加恢复的存储点名称
    #recovery_target_timeline = 'latest' # 原来做全备恢复的配置，切记注释掉，不然启动数据库时会一直提示无效的主检查点
    [postgres@db01 ~]$ pg_ctl start -l start.log  # 正常启动数据库
    postgres=# select pg_wal_replay_resume(); # 执行启动恢复操作
    ```

  - 查看恢复结果

    ```
    postgres=# \c testdb01
    You are now connected to database "testdb01" as user "postgres".
    testdb01=# \d
            List of relations
     Schema | Name | Type  |  Owner
    --------+------+-------+----------
     public | t1   | table | postgres
     public | t2   | table | postgres
    (2 rows)
    
    testdb01=# select count(*) from t1;
     count
    -------
         8
    (1 row)
    
    testdb01=# select count(*) from t2;
     count
    -------
         1
    (1 row)
    testdb01=# select * from t2;
     id | name
    ----+------
      2 | b
    (1 row)
    ```

#### 基于LSN号的不完全恢复

+ 获取对象的OID

  ```
  testdb01=# select oid from pg_class where relname='t1';
    oid
  -------
   16482
  (1 row)
  ```

+ 获取当前wal日志文件

  ```
  postgres=# select pg_walfile_name(pg_current_wal_lsn());
       pg_walfile_name
  --------------------------
   00000006000000000000000E
  (1 row)
  ```

+ 使用`pg_waldump`分析日志文件，找到LSN号

  ```
  pg_waldump WAL_FILE_NAME >> log.mnr
  ```

+ waldump出来的日志简述

  ```
  rmgr: Heap        len (rec/tot):     54/   330, tx:        828, lsn: 0/0D000060, prev 0/0D000028, desc: INSERT off 7 flags 0x00, blkref #0: rel 1663/16468/16493 blk 0 FPW
  
  rmgr:Heap:表示这条语句是DML操作
  tx:        828:XID，事务ID
  lsn: 0/0D000060:表示LSN号
  INSERT off:表示执行的是INSERT语句
  rel 1663/16468/16493:relation 表空间OID/数据库OID/表的OID
  ```

+ 选LSN号的原则

  ```
  LSN号可能有许多个，选择同一个XID(事务ID)内的所有LSN号都可以
  ```

+ 修改`postgres.conf`配置

  ```
  设置restore_command和recovery_target_lsn参数：
  resrore_command = 'cp /home/postgres/arch_log/%f %p';
  recovery_target_name = '0/0D000060';
  ```

+ 在数据目录创建标记文件

  ```
  [postgres@db01 ~]$ touch $PGDATA/recovery.signal
  ```

+ 正常启动数据库恢复数据

  ```
  [postgres@db01 ~]$ pg_ctl start -l start.log
  ```

+ 执行存储过程，启动新的timeline，使数据库处于可读可写状态

  ```
  postgres=# select pg_wal_replay_resume();
  ```

#### 基于XID(事务号)的不完全恢复

+ 同基于存储点和基于LSN号恢复，`postgres.conf`配置为

  ```
  设置restore_command和recovery_target_lsn参数：
  resrore_command = 'cp /home/postgres/arch_log/%f %p';
  recovery_target_xid = '828';
  ```

#### 基于时间点的不完全恢复

+ 同上，`postgres.conf`配置为

  ```
  设置restore_command和recovery_target_lsn参数：
  resrore_command = 'cp /home/postgres/arch_log/%f %p';
  recovery_target_time = 'TIMESTAMP';
  ```

  

