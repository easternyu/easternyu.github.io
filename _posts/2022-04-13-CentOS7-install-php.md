---
title: CentOS7编译安装PHP
date: 2022-04-13 00:00:00
categories:
- learn/learn
tags:
---



### CentOS7编译安装PHP

#### 1.安装依赖包

```
[root@testlab ~]# yum install libxml2-devel sqlite-devel -y
```

#### 2.创建用户

```
[root@testlab ~]# groupadd www-data
[root@testlab ~]# useradd -g www-data www-data
```

#### 2.下载并解压源码包

```
[root@testlab ~]# wget -O /usr/local/src/php.tar.gz https://www.php.net/distributions/php-7.4.21.tar.gz
[root@testlab ~]# mkdir -pv /usr/local/src/php
[root@testlab ~]# tar -zxvf /usr/local/src/php.tar.gz -C /usr/local/src/php/ --strip-components 1
```

#### 3.编译安装

```
[root@testlab ~]# cd /usr/local/src/php
[root@testlab php]# ./configure --prefix=/usr/local/php --enable-fpm
[root@testlab php]# make && make install
```

#### 4.复制相关配置文件并修改参数

```
[root@testlab ~]# cp /usr/local/src/php/php.ini-production /usr/local/php/lib/php.ini
[root@testlab ~]# cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
[root@testlab ~]# ln -sv /usr/local/php/sbin/php-fpm /usr/local/bin
[root@testlab ~]# cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
[root@testlab ~]# sed -i 's@;cgi.fix_pathinfo=1@cgi.fix_pathinfo=0@;s@;pid = run/php-fpm.pid@pid = /run/php-fpm.pid@' /usr/local/php/lib/php.ini
[root@testlab ~]# sed -i 's@user = nobody@user = www-data@;s@group = nobody@group = www-data@' /usr/local/php/etc/php-fpm.d/www.conf 
```

#### 5.启动

```
[root@testlab ~]# php-fpm
```

#### 6.修改nginx配置并重启nginx服务

```
[root@testlab ~]# vim /usr/local/nginx/conf/nginx.conf #增加以下内容：
        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            include        fastcgi_params;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  SCRIPE_NAME      $fastcgi_script_name;
        }
[root@testlab ~]# systemctl restart nginx
```

#### 附录：复杂一些的编译选项和依赖安装

+ 编译选项

  ```
  [root@testlab ~]# ./configure --prefix=/usr/local/php --with-curl --with-freetype \
              --enable-gd --with-gettext --with-iconv-dir --with-kerberos --with-libdir=lib64 \
              --with-mysqli --with-openssl --with-pdo-mysql \
              --with-pdo-sqlite --with-pear --with-jpeg --with-xmlrpc \
              --with-xsl --with-zlib --with-bz2 --with-mhash --enable-fpm --enable-bcmath \
              --enable-inline-optimization --with-fpm-systemd \
              --enable-mbregex --enable-mbstring --enable-opcache --enable-pcntl \
              --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --enable-sysvshm \
              --enable-xml --with-zip
  ```

+ 需要的依赖

  + 可以使用yum和rpm安装的依赖

    ```
    [root@testlab ~]# yum install libxml2-devel autoconf sqlite-devel systemd-devel libpng-devel libjpeg-devel freetype-devel bzip2-devel libxslt-devel libcurl-devel -y
    [root@testlab ~]# rpm -ivh http://mirrors.163.com/centos/7.9.2009/cloud/x86_64/openstack-queens/Packages/o/oniguruma-6.7.0-1.el7.x86_64.rpm
    [root@testlab ~]# rpm -ivh http://mirrors.163.com/centos/7.9.2009/cloud/x86_64/openstack-queens/Packages/o/oniguruma-devel-6.7.0-1.el7.x86_64.rpm
    ```

  + 需要手动编译安装的依赖

    ```
    libzip-devel >= 0.10需要编译安装
    先安装CMake
    [root@testlab ~]# yum install gcc-c++ -y
    [root@testlab ~]# wget https://cmake.org/files/v3.21/cmake-3.21.1.tar.gz
    [root@testlab ~]# tar -zxvf cmake-3.21.1.tar.gz
    [root@testlab ~]# cd cmake-3.21.1
    [root@testlab cmake-3.21.1]# ./bootstrap --prefix=/usr/local/cmake
    [root@testlab cmake-3.21.1]# make
    [root@testlab cmake-3.21.1]# make install
    [root@testlab cmake-3.21.1]# ln -sv /usr/local/cmake/bin/cmake /usr/bin/cmake
    再编译安装libzip
    [root@testlab ~]# wget https://libzip.org/download/libzip-1.8.0.tar.gz
    [root@testlab ~]# tar -zxvf libzip-1.8.0.tar.gz
    [root@testlab libzip-1.8.0]# cd libzip-1.8.0
    [root@testlab libzip-1.8.0]# mkdir -pv build && cd build && cmake .. && make && make install
    [root@testlab libzip-1.8.0]# export PKG_CONFIG_PATH='/usr/local/lib64/pkgconfig/'
    ```
