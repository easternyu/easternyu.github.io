---
title: Oracle实例和监听的启动和关闭
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---


### Oracle实例和监听的启动和关闭 

#### 实例的启动和关闭

  - 实例的启动

  ```sql
  SQL> startup;
  ```
  - 实例的关闭

  ``` 
  SQL> shutdown;
  或者:
  SQL> shutdown normal; --正常关闭，会等待所有会话退出后才关闭，使用较少
  SQL> shutdown immediate; --快速关闭，回滚所有没有提交的事务，关闭所有会话，最常使用的关闭方式
  SQL> shutdown abort; --强制关闭，强制断开所有会话，不对事务做处理，需要实例恢复，如无特殊情况不建议使用
  ```

#### 监听的启动和关闭

  - 查看监听状态

  ```
  [oracle@db01 ~]$ lsnrctl status
  ```

  - 监听的启动

  ```
  [oracle@db01 ~]$ lsnrctl start
  
  ```

  - 监听的关闭 

  ```
  [oracle@db01 ~]$ lsnrctl stop
  ```

  - 监听启动后动态注册的问题

  ```
  动态注册的监听在'lsnrctl start'之后通常不会马上注册上，需要等待1分钟左右，或者手动注册:
  SQL> alter system register;
  ```

  
