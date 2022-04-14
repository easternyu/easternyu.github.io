---
title: Oracle备份恢复-简单的备份
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---



### Oracle12C备份恢复2-简单的备份

#### 非归档模式

1. 重启数据库到mount状态

   ```
   SQL> shutdown immediate;
   SQL> startup mount;
   ```

2. 备份

   ```
   [oracle@db01 ~]$ rman target /
   RMAN> backup database;
   或者
   RMAN> backup tag 'full_db_bak' format '/u01/backup/db_%U' database;
   tag：设置备份标记
   format：设置备份路径和文件名，%U表示生成唯一文件名
   ```

#### 归档模式

1. 开启归档模式

2. 备份

   ```
   [oracle@db01 ~]$ rman target /
   RMAN> backup database;
   ```

3. 如果备份中包含system表空间的文件，则自动备份控制文件和spfile

#### 备份数据库同时备份归档文件

```
RMAN> backup database plus archivelog;
```


