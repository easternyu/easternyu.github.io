---
title: Oracle实例的关闭过程
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---

### Oracle实例的关闭过程

#### open->close(mount)

`SQL> alter database close;`

1. 正常关闭的情况：将内存中的数据写入到数据文件和redo日志文件中，关闭数据文件和redo日志文件，数据库处于mount状态，但控制文件依旧是打开的状态。
2. 非正常关闭的情况：使用`SQL> shutdown abort;`或者断电等情况下，没来得及将内存中的数据写入到数据文件和redo日志文件中，下次实例启动时，Oracle会进行实例恢复

### close->nomount

`SQL> alter database dismount;`

关闭控制文件，但实例仍然存在内存中

### nomount->shutdown

`SQL> shutdown;`

实例关闭，后台进程终止

在某些情况下，实例无法干净的关闭，无法重新启动实例，这时候可以使用`SQL> shutdown abort;`先关闭

#### 实例关闭的四种模式

|              | Abort | immediate | transactional | normal |
| -------------------- | ----- | --------- | ------------- | ------ |
| 允许新连接           | NO    | NO        | NO            | NO     |
| 等待当前会话结束     | NO    | NO        | NO            | YES    |
| 等待当前事务结束     | NO    | NO        | YES           | YES    |
| 执行检查点并关闭文件 | NO    | YES       | YES           | YES    |

shutdown abort：使用此模式和断电一样的效果，速度最快，但下次启动时需要进行实例恢复，这种恢复时Oracle自动进行的

shutdown immediate：此模式关闭速度仅次于abort，断开用户连接，回滚未提交的事务

shutdown transactional：等待当前所有事物结束才关闭数据库

shutdown normal：等待所有用户断开连接才关闭数据库，默认模式

#### 生产中关闭数据库的方法

1. 将内存中的数据刷到数据文件中：`SQL> alter system flush buffer_cache;`

2. 执行检查点：`SQL> alter system checkpoint;`

3. `SQL> shutdown immediate;`

4. `SQL> shutdown abort;` --immediate方式不能正常关闭的情况下使用
