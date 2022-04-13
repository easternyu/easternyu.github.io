---
title: CentOS7升级SSH
date: 2022-04-13 00:00:00
categories:
- learn/learn
tags:
---







### CentOS7升级SSH

#### 1.环境信息

```
[root@localhost ~]# cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
[root@localhost ~]# uname -a
Linux localhost.localdomain 3.10.0-1160.el7.x86_64 #1 SMP Mon Oct 19 16:18:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
[root@localhost ~]# rpm -qa | wc -l
340
[root@localhost ~]# yum repolist
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.163.com
 * extras: mirrors.163.com
 * updates: mirrors.163.com
repo id                             repo name                             status
base/7/x86_64                       CentOS-7 - Base                       10,072
extras/7/x86_64                     CentOS-7 - Extras                        498
updates/7/x86_64                    CentOS-7 - Updates                     2,542
repolist: 13,112
```

#### 2.下载并解压所需源码包

+ zlib

  ```
  [root@localhost src]# wget https://nchc.dl.sourceforge.net/project/libpng/zlib/1.2.11/zlib-1.2.11.tar.gz
  [root@localhost src]# tar -zxvf zlib-1.2.11.tar.gz
  ```

+ openssl

  ```
  [root@localhost src]# wget https://github.com/openssl/openssl/archive/refs/tags/OpenSSL_1_1_1k.tar.gz
  [root@localhost src]# tar -zxvf openssl-1.1.1k.tar.gz
  ```

+ openssh

  ```
  [root@localhost src]# wget https://openbsd.hk/pub/OpenBSD/OpenSSH/portable/openssh-8.6p1.tar.gz
  [root@localhost src]# tar -zxvf openssh-8.6p1.tar.gz
  ```

#### 3.编译升级zlib

```
[root@localhost zlib-1.2.11]# ./configure --prefix=/usr/local/zlib
[root@localhost zlib-1.2.11]# make
[root@localhost zlib-1.2.11]# make install
```

#### 4.编译升级openssl

```
[root@localhost openssl-1.1.1k]# ./config --prefix=/usr/local/openssl
[root@localhost openssl-1.1.1k]# ./config -t
[root@localhost openssl-1.1.1k]# make test #建议测试，因为中间可能会报错
[root@localhost openssl-1.1.1k]# make #处理完报错之后再进行make
[root@localhost openssl-1.1.1k]# make install
[root@localhost openssl]# ldd /usr/local/openssl/bin/openssl
[root@localhost openssl]# echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
[root@localhost openssl]# ldconfig -v
[root@localhost openssl]# which openssl
/usr/bin/openssl
[root@localhost openssl]# mv /usr/bin/openssl /usr/bin/openssl.old
[root@localhost openssl]# ln -sv /usr/local/openssl/bin/openssl /usr/bin/openssl
‘/usr/bin/openssl’ -> ‘/usr/local/openssl/bin/openssl’
[root@localhost openssl]# openssl version
OpenSSL 1.1.1k  25 Mar 2021
```

+ 报错一

  ```
  Can't locate Test/Harness.pm in @INC (@INC contains: /usr/local/src/openssl-1.1.1k/test/../util/perl /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 .) at .././test/run_tests.pl line 112.
  BEGIN failed--compilation aborted at .././test/run_tests.pl line 112.
  make[1]: *** [_tests] Error 2
  make[1]: Leaving directory `/usr/local/src/openssl-1.1.1k'
  make: *** [tests] Error 2
  ```

  解决：

  ```
  [root@localhost openssl-1.1.1k]# yum install perl-Test-Harness -y
  ```

+ 报错二

  ```
  Parse errors: No plan found in TAP output
  Files=158, Tests=0,  1 wallclock secs ( 0.31 usr  0.12 sys +  0.81 cusr  0.39 csys =  1.63 CPU)
  Result: FAIL
  make[1]: *** [_tests] Error 1
  make[1]: Leaving directory `/usr/local/src/openssl-1.1.1k'
  make: *** [tests] Error 2
  ```

  解决：

  ```
  [root@localhost openssl-1.1.1k]# yum install perl-CPAN -y
  [root@localhost openssl-1.1.1k]# perl  -MCPAN  -e  shell 
  # 中间需要确认的地方一路yes回车，进入CPAN的shell界面
  cpan[1]> install Text::Template
  ```

#### 5.编译升级openssh

```
[root@localhost openssh-8.6p1]# mkdir -pv /root/ssh_bak
[root@localhost openssh-8.6p1]# mv /etc/ssh/* /root/ssh_bak
[root@localhost openssh-8.6p1]# ./configure --prefix=/usr/ --sysconfdir=/etc/ssh/ --with-md5-passwords --with-ssl-dir=/usr/local/openssl/ --with-zlib=/usr/local/zlib/ --without-hardening
[root@localhost openssh-8.6p1]# make
[root@localhost openssh-8.6p1]# make install
[root@localhost openssh-8.6p1]# mv /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
[root@localhost openssh-8.6p1]# cp /root/ssh_bak/sshd_config /etc/ssh/sshd_config
[root@localhost openssh-8.6p1]# cp contrib/redhat/sshd.init /etc/init.d/sshd
[root@localhost openssh-8.6p1]# chmod +x /etc/init.d/sshd
[root@localhost openssh-8.6p1]# mv /usr/lib/systemd/system/sshd.service /usr/lib/systemd/system/sshd.service.bak
[root@localhost openssh-8.6p1]# cat << EOF >> /usr/lib/systemd/system/sshd.service
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target
[Service]
ExecStart=/usr/sbin/sshd
[Install]
WantedBy=multi-user.target
EOF
[root@localhost openssh-8.6p1]# sed -i 's@#PermitRootLogin yes@PermitRootLogin  yes@;s@GSSAPIAuthentication yes@#GSSAPIAuthentication yes@;s@GSSAPICleanupCredentials yes@#GSSAPICleanupCredentials yes@;s@UsePAM yes@#UsePAM yes@' /etc/ssh/sshd_config
[root@localhost openssh-8.6p1]# setenforce 0
[root@localhost openssh-8.6p1]# systemctl daemon-reload
[root@localhost openssh-8.6p1]# systemctl restart sshd
[root@localhost openssh-8.6p1]# sshd --help
unknown option -- -
OpenSSH_8.6p1, OpenSSL 1.1.1k  25 Mar 2021
[root@localhost ~]# ssh -V
OpenSSH_8.6p1, OpenSSL 1.1.1k  25 Mar 2021
```


