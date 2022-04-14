---
title: Oracle的密码文件
date: 2022-04-14 00:00:00
categories:
- learn/learn
tags:
---



### Oracle的密码文件

#### 密码文件的存放路径
```
Linux:$ORACLE_HOME/dbs/orapwSID
Windows:$ORACLE_HOME/database/PWDORACLE_SID.ora
```

#### 密码文件读取顺序

```
orapwSID->orapw
```

#### sys用户忘记密码的处理方法

  - 通过操作系统验证直接修改

  ```
  [oracle@db01 ~]$ sqlplus / as sysdba
  SQL> alter user sys identified by Oracle123;
  ```

  - 重建密码文件

  ```
  [oracle@db01 ~]$ orapwd file=$ORACLE_HOME/dbs/orapwSID password=Oracle123 force=y
  ```

####  相关参数
`remote_login_passwordfile`

#### 相关视图
`v$pwfile_user;`
