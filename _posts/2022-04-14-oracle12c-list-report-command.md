---
title: Oracle备份恢复-list-report命令
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---

### Oracle12C备份恢复-list-report命令

#### list命令

1. 列出所有备份信息

   ```
   RMAN> list backup;
   或者
   RMAN> list bakcupset;
   ```

2. 列出包含指定数据文件的备份集

   ```
   RMAN> list backup of datafile 1;
   ```

3. 列出指定备份集

   ```
   RMAN> list backupset 1;
   ```

4. 列出指定tab的备份集

   ```
   RMAN> list backupset tag 'TAG20210619T145551';
   ```

5. 列出所有的归档日志

   ```
   RMAN> list archivelog all;
   ```

6. 列出包含指定表空间的备份集

   ```
   RMAN> list backupset of tablespace users;
   ```

7. 按文件列出所有备份集

   ```
   RMAN> list bakcupset by file;
   ```

8. 列出备份集的简略信息

   ```
   RMAN> list backupset summary;
   ```

9. 列出数据库的备份集

   ```
   RMAN> list backupset of database; --实际效果等同于list backupset;
   ```


#### report命令

1. 查看构成数据库的文件

   ```
   RMAN> report schema;
   ```

2. 查看哪些文件需要备份

   ```
   RMAN> report need backup;
   ```

3. 查看哪些文件上执行了不可恢复的操作（比如使用no logging方式插入数据）

   ```
   RMAN> report unrecoverable;
   ```

4. 列出指定天数未备份的文件

   ```
   RMAN> report need backup days 3; -- 列出三天内未备份的文件
   ```

5. 列出没有指定备份个数的文件

   ```
   RMAN> report need backup redundancy 3; -- 列出没有三个备份的文件
   ```

6. 列出指定表空间是否需要备份

   ```
   RMAN> report need backup tablespace users; -- 列出users表空间是否需要备份
   ```

7. 列出违反保留策略的备份集（可以说是无效备份）

   ```
   RMAN> report obsolete;
   ```

   
