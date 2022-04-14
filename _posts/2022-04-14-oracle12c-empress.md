---
title: Oracle EM的基本操作
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---


### EM Express的启用和关闭

  - 查询EM Express的状态

  ```
  SQL> select dbms_xdb_config.gethttpport from dual;  --查看http方式的EM状态，结果0表示不启用EM Express，结果为其他数字表示EM Express监听在此端口上
  SQL> select dbms_xdb_config.gethttpsport from dual;  --查看https方式的EM状态，同http方式
  ```

  - 启用EM Express

  ```
  SQL> exec dbms_xdb_config.sethttpport(5505); --在5505端口上启用http方式的EM Express，5505端口可以替换成其他自定义端口
  SQL> exec dbms_xdb_config.sethttpsport(5500); --在5500端口上启用https方式的EM Express，5500端口可以替换成其他自定义端口
  ```

  - 停用EM Express

  ```
  SQL> exec dbms_xdb_config.sethttpport(0); --停用http方式的EM Express
  SQL> exec dbms_xdb_config.sethttpsport(0); --停用https方式的EM Express
  ```

  


