---
title: Oracle的表空间
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---



### Oracle的表空间

#### 表空间的管理方式

1. 区管理方式：本地管理和字段管理
2. 段管理方式：自动管理和手工管理
3. system、temp、undo表空间的段使用手工管理，其他为自动管理

#### 表空间的类类型

1. 永久表空间
2. 临时表空间
3. undo表空间

#### 表空间的操作

1. 创建永久表空间

   ```
   SQL> create tablespace tbs01 datafile '/u01/app/oracle/oradata/prod/tbs01_1.dbf' size 100m autoextend on next size 100m extent management local uniform size 1m segment space management auto;
   ```

2. 创建bigfile表空间

   ```
   SQL> create bigfile tablespace tbs02 datafile '/u01/app/oracle/oradata/prod/tbs02.dbf' size 100m;
   ```

3. 创建临时表空间

   ```
   SQL> create temporary tablespace temptbs01 tempfile '/u01/app/oracle/oradata/prod/temptbs01_1.dbf' size 100m;
   ```

4. 创建undo表空间

   ```
   SQL> create undo tablespace undotbs01 datafile '/u01/app/oracle/oradata/prod/undotbs01_1.dbf' size 100m;
   ```

5. 将新建的undo表空间设置为默认

   ```
   SQL> alter system undo_tablespace undotbs01;
   ```

6. 将新建的临时表空间加入表空间组

   ```
   SQL> alter temporary tablesapce temptbs01 tablespace group GROUP1;
   ```

7. 表空间增加大小

   ```
   1.向表空间增加数据文件方式：
   SQL> alter tablespace tbs01 add datafile '/u01/app/oracle/oradata/prod/tbs01_2.dbf' size 100m; --永久表空间
   SQL> alter tablesapce temptbs01 add tempfile '/u01/app/oracle/oradata/prod/temptbs01_2.dbf' size 100m;  --数据表空间
   SQL> alter tablespace undotbs01 add datafile '/u01/app/oracle/oradata/prod/undotbs01_2.dbf' size 100m; --undo表空间
   2.直接扩大原来的数据文件
   SQL> alter database datafile 'u01/app/oracle/oradata/prod/tbs01_1.dbf' resize 500m;
   ```

8. 删除表空间

   ```
   SQL> drop tablespace tbs01; --删除空的表空间
   SQL> drop tablespace tbs01 including contents; --删除表空间并且删除其中的内容
   QSL> drop tablespace tbs02 including contents and datafiles; --删除表空间斌且删除其中的内容和所有数据文件
   ```

9. 表空间的online/offline操作

   ```
   SQL> alter tablespace tbs1 offline;
   SQL> alter tablespace tbs1 online;
   ```

10. 表空间的只读/读写操作

    ```
    SQL> alter tablespace tbs1 read only;
    SQL> alter tablesapce tbs1 read write;
    ```


