---
title: Oracle的undo数据
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---


### 管理Oracle的undo数据

当某个进程更改了数据库中的数据时，Oracle会保存旧值（还原数据）。按数据修改前的原样存储数据，如果捕获了还原数据，则可以回退未提交的数据。还原数据还用于读一致性和闪回查询

#### 相关参数

1. undo_management：undo的管理方式
2. undo_retention：undo数据的保留时间
3. undo_tablespace：正在使用的undo表空间

#### 相关视图

1. dba_rollback_segs
2. v$rollname
3. dba_undo_extents

#### undo表空间的操作

参考：[Oracle12C基础学习9-表空间](https://blog.19920413.xyz/index.php/archives/11/)
