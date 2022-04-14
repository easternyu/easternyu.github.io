---
title: Oracle最简单的用户创建与使用
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---


### Oracle基础学习番外1-最简单的用户创建与使用

```
SQL> create user tuser01 identified by tuser01; --创建用户tuser01密码也是tuser01
SQL> grant resource,connect to tuser01; --赋予用户tuser01使用资源和连接数据库的权限
SQL> alter user tuser01 quota unlimited on user; --赋予用户tuser01可以无限制使用users表空间
SQL> conn tuser01/tuser01 --连接到tuser01用户
SQL> create table test1(id int); --创建一个测试表
SQL> insert into test1 values(0); --插入一条测试数据
SQL> commit; --提交数据完成操作
```


