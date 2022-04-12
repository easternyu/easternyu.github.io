---
title: CentOS7编译安装nginx
date: 2022-04-12 00:00:00
categories:
- learn/learn
tags:
---



### CentOS7编译安装nginx

#### 1.安装依赖包

```
[root@testlab ~]# yum install gcc pcre-devel openssl-devel -y
```

#### 2.创建用户

```
[root@testlab ~]# useradd -s /sbin/nologin -M nginx
```

### 3.创建缓存文件夹并赋予权限

```
[root@testlab ~]# mkdir -pv /var/cache/nginx
[root@testlab ~]# chown -R nginx. /var/cache/nginx
```

#### 4.下载并解压源码包

```
[root@testlab ~]# wget -O /usr/local/src/nginx.tar.gz http://nginx.org/download/nginx-1.20.1.tar.gz
[root@testlab ~]# mkdir -pv /usr/local/src/nginx
[root@testlab ~]# tar -zxvf /usr/local/src/nginx.tar.gz -C /usr/local/src/nginx/ --strip-components 1
```

#### 5.编译安装

```
[root@testlab ~]# cd /usr/local/src/nginx
[root@testlab nginx]# ./configure --prefix=/usr/local/nginx \
            --error-log-path=/var/log/nginx/error.log \
            --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid \
            --lock-path=/var/run/nginx.lock \
            --http-client-body-temp-path=/var/cache/nginx/client_temp \
            --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
            --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
            --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
            --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx \
            --with-compat --with-file-aio --with-threads --with-http_addition_module \
            --with-http_auth_request_module --with-http_dav_module --with-http_flv_module \
            --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module \
            --with-http_random_index_module --with-http_realip_module \
            --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module \
            --with-http_stub_status_module --with-http_sub_module --with-http_v2_module \
            --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module \
            --with-stream_ssl_module --with-stream_ssl_preread_module \
            --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' \
            --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
[root@testlab nginx]# make && make install
```

#### 6.写入systemd服务文件

```
[root@testlab ~]# cat << EOF >> /usr/lib/systemd/system/nginx.service
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/bin/sh -c "/bin/kill -s HUP $(/bin/cat /var/run/nginx.pid)"
ExecStop=/bin/sh -c "/bin/kill -s TERM $(/bin/cat /var/run/nginx.pid)"

[Install]
WantedBy=multi-user.target
EOF
```

#### 7.启动并放开防火墙限制

```
[root@testlab ~]# systemctl daemon-reload
[root@testlab ~]# systemctl start nginx
[root@testlab ~]# firewall-cmd --permanent --add-service=http
[root@testlab ~]# firewall-cmd --reload
```


