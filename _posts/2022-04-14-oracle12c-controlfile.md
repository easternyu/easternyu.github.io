---
title: Oracle的控制文件
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---



### Oracle的控制文件

#### 控制文件包含的信息

  - 数据库名
  - 数据库创建的时间戳
  - 数据文件、redo日志文件、归档日志文件位置
  - 表空间信息
  - RMAN备份信息

#### 控制文件的作用
  1. 控制文件含有数据文件、redo日志文件等位置信息，数据库打开会用到这个信息。当数据库增加、重命名、删除了文件，会更新控制文件。
  2. 包含一些元数据，在数据库未open之前，控制文件里面包含有检查点checkpoint信息，当数据库需要恢复时需要这个信息，每三秒钟检查点进程ckpt会记录检查点到控制文件

#### 多份控制文件的意义

  - 控制文件至少有一份，一般都是两份或更多，多份控制文件内容完全相同，并且同时被Oracle打开
  - 避免单点故障
  - 一般放置在多个不同磁盘（如果有多个磁盘）位置

#### 控制文件结构

  * 由section组成，每个section由record组成

  * dump控制文件：

  ```
  SQL> alter session set events 'immediate trace name controlf level 1'; --level 1如果不够详细，可以使用level 3或者其他level
  找到这个dump出来的文件：
  SQL> select name,value from v$diag_info; --找到name为Default Trace File的value，这个值就是当前trace文件的位置
  ```

  * record记录的分类
    * 循环重用记录

      ```
      这些记录包含可以被覆盖的非关键信息。当所有可用的记录槽用完时，数据库需要扩展控制文件或覆盖最旧的记录，以便为新记录腾出空间。循环重用记录可以删除，并且不会影响数据库运行，如：RMAN备份记录，归档日志历史信息等信息。
      ```
    * 非循环重用记录

      ```
      这些记录包含不经常更改且不能被覆盖的关键信息。包括表空间、数据文件、联机重做日志文件、redo线程。Oracle数据库绝不会重用这些记录，除非从表空间中删除相应的对象。
      ```

#### 相关视图和参数

  - 参数：

  ```
  control_file_record_keep_time：循环重用记录最小保留天数，范围是0-365天
  修改：SQL> alter system set control_file_record_keep_time=30; --修改保留时间为30天
  ```

  - 视图
    - v$controlfile
    - v$controlfile_record_section

#### 控制文件的增加、减少、修改

  - 增加控制文件数量：
    - 方法1：修改参数、关闭数据库、复制原有控制文件成为新的控制文件、启动数据库

      ```
      SQL> alter system set control_files='/PATH/TO/controlfile01','/PATH/TO/controlfile02','/PATH/TO/controlfile03' scope=spfile;
      SQL> shutdown immediate;
      [oracle@db01 ~]$ cp /PATH/TO/control01 /PATH/TO/control03
      SQL> startup;
      ```
    - 方法2：使用spfile创建pfile、修改pfile、关闭数据库、使用pfile创建spfile、启动数据库
    
      ```
      具体过程略
      ```
  - 减少控制文件数量、修改控制文件位置、修改控制文件名称同理

#### 清理控制文件记录

  - 重建控制文件或者设置control_file_record_keep_time=0（较少使用，有风险）

  - 使用包（sys.dbms_backup_restore.resetCfileSection）清理：

  ```
  SQL> execute sys.dbms_backup_restore.resetCfileSection(9); --清理v$log_history对应记录
  SQL> execute sys.dbms_backup_restore.resetCfileSection(11); --清理v$archived_log对应记录
  SQL> execute sys.dbms_backup_restore.resetCfileSection(28); --清理v$rman_status对应记录
  SQL> execute sys.dbms_backup_restore.resetCfileSection(12); --清理RMAN备份信息
  ```
