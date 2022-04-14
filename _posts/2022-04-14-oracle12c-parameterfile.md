---
title: Oracle的参数文件
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---


### 参数文件

#### 参数文件的分类

  * pfile：文本格式的参数文件
  * spfile：二进制格式的参数文件

#### 参数文件的命名规则

  * pfile：initSID.ora
  * spfile：spfileSID.ora

#### 启动实例时读取参数文件的顺序

  - 在$ORACLE_HOME/dbs目录下先找spfile：spfileSID.ora或者spfile.ora
  - 再找pfile：initSID.ora

#### 修改参数

  - 静态参数：重启数据库生效

  ```
  SQL> alter system set processes=500 scope=spfile;  -- 修改范围指定spfile，下次启动数据库生效
  ```

  - 动态参数：立即生效
    - 会话参数：影响当前会话

      ```
      SQL> alter session set nls_date_format='yyyy-mm-dd hh24:mi:ss';
      ```
    - 系统参数：影响数据库和所有会话
    
      ```
      SQL> alter system set memory_target=4G;
      ```

#### pfile和spfile的互相创建

  - 基于spfile创建pfile

  ```
  SQL> create pfile from spfile;
  ```

  - 基于pfile创建spfile

  ```
  SQL> create spfile from pfile;
  ```

#### 使用pfile启动数据库
```
SQL> startup pfile='/PATH/TO/initSID.ora';
```


