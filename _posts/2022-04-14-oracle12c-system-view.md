---
title: Oracle系统视图
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---





### Oracle系统视图

#### 静态数据字典视图

```
分为三类，对应三个前缀
user_*：该视图存储了关于当前用户所拥有的对象的信息
all_*：该视图存储了当前用户能够访问的对象的信息
dba_*：该视图存储了数据库中所有对象的信息
```

#### 动态性能视图
```
以v$或者gv$开头的视图，v$是单实例的信息，gv$是rac环境下的信息
```

#### 查看所有数据字典
```
SQL> select * from dictionary;
或者
SQL> select * from dict;
```

#### 授予一个普通用户查询所有数据字典权限
```
SQL> grant select any dictionary to hr;
```

#### 常用视图
```
v$version：数据库版本信息
v$database：数据库信息
v$instance：数据库实例信息
v$session：连接会话信息
v$tablespace,dba_tablespaces：表空间信息
v$datafile,v$dbfile,dba_data_files：数据文件信息
v$tempfile,dba_temp_files：临时文件信息
dba_tables：所有表信息
dba_indexes：所有索引信息
dba_views：所有视图信息
v$log,v$logfile：redo日志文件信息
v$controlfile：控制文件信息
```


