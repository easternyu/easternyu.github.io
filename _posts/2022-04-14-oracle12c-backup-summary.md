---
title: Oracle备份恢复概述
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---




### Oracle备份恢复1-备份恢复概述

#### 备份恢复类型

1. 物理备份
    RMAN

2. 逻辑备份
    导入导出工具：exp/imp
    数据泵工具：expdp/impdp

#### 使用RMAN

1. 服务端

   ```
   [oracle@db01 ~]$ rman target /
   或者
   [oracle@db01 ~]$ rman target / nocatalog
   ```

2. 客户端

   ```
   C:\Users\Administrator>rman target sys/Oracle123@10.21.100.250:1521/prod
   ```

   RMAN链接的账户必须有sysdba权限

#### 归档

1. 查看是否处于归档模式

   ```
   SQL> archive log list;
   或者
   SQL> select log_mode from v$database;
   ```

2. 打开归档模式

   1. 设置参数

      ```
      SQL> alter system set log_archive_dest='/u01/archive' scope=spfile; --设置归档存放目录
      SQL> alter system set log_archive_format='%t_%s_%r.arc' scope=spfile; --设置归档文件格式
      %t:归档线程号
      %s:日子序列号
      %r:resetlogs ID
      %d:database ID
      ```

   2. 重启数据库到mount，打开归档并打开数据库

      ```
      SQL> shutdown immediate;
      SQL> startup mount;
      SQL> alter database archivelog;
      SQL> alter database open;
      ```

3. 关闭归档模式

   ```
   SQL> shutdown immediate;
   SQL> startup mount;
   SQL> alter database noarchivelog;
   ```

#### 一致性备份

​    当数据库处于一致性状态的时候备份

​    当数据库关闭状态处于一致性状态

​    脱机备份

#### 非一致性备份

​    当数据库处于非一致性状态的时候备份

​    实例失败，`shutdown abort`

​    数据库打开的时候进行备份是非一致性备份

​    处于归档模式

#### 相关参数
    control_file_record_keep_time：备份信息在控制文件中保存的时间，循环覆盖，默认为7天，一般建议修改成30天
