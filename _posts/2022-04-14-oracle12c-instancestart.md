---
title: Oracle实例的启动过程
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---



### Oracle实例的启动过程

#### 实例的四个阶段

1. shutdown
2. nomount
3. mount
4. open

#### shutdown->nomount

`SQL> startup nomount;`

1. 寻找初始化参数文件
    spfileSID.ora->spfile.ora->initSID.ora

2. 读取参数文件的参数

3. 根据参数分配sga大小

4. 启动后台进程

5. 将所有显示的参数写入alter日志

### nomount->mount

`SQL> alter database mount;`

数据库挂在，获取控制文件（控制文件位置由参数control_files控制），读取控制文件获取数据文件和redo日志文件位置。

在mount状态，数据库并未打开，但是此时客户端可以连上监听。

#### mount->open

`SQL> alter database open;`

只有数据库open，用户才能对数据进行操作，例如增删改查操作，exp导出操作等

1. 先打开除了undo表空间外的表空间数据文件
    如果表空间是离线的，那么数据文件也会离线

2. 再打开undo表空间文件
    如果有多个undo表空间，则会根据参数undo_tablespace决定使用哪个

