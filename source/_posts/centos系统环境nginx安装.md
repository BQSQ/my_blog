---
title: centos系统环境nginx安装
date: 2018-03-03 17:57:56
tags: [nginx,安装,centos]
categories: [nginx]
keywords: [nginx,nginx安装,centos,zlib.pcre]
---

确认该系统已经安装过gcc编译器,centos安装nginx需要安装pcre、zlib库。   
[ngixn官方下载地址](http://nginx.org/en/download.html)   
[nginx1.12.2.tar.gz](http://opvqbxg2k.bkt.clouddn.com/nginx/nginx-1.12.2.tar.gz)   
[pcre-8.39.tar.gz](http://opvqbxg2k.bkt.clouddn.com/nginx/pcre-8.39.tar.gz)   
[zlib-1.2.8.tar.gz](http://opvqbxg2k.bkt.clouddn.com/nginx/zlib-1.2.8.tar.gz)

<!-- more -->

1. 安装PCRE库

``` shell
cd /usr/local/
#将pcre文件拷贝到该文件夹下
tar -zxvf pcre-8.21.tar.gz 
cd pcre-8.21 
./configure
make
make install
```

2. 安装zlib库

``` shell
cd /usr/local/  
#将zlib文件拷贝到该文件夹下
tar -zxvf zlib-1.2.8.tar.gz
cd zlib-1.2.8
./configure
make
make install
```

3. 安装nginx

``` shell
cd /usr/local/
#将nginx文件拷贝到该文件夹下
tar -zxvf nginx-1.12.2.tar.gz 
cd nginx-1.12.2
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_gzip_static_module --with-http_stub_status_module --with-pcre=/usr/local/pcre-8.39 --with-zlib=/usr/local/zlib-1.2.8 --with-http_ssl_module
```

4. nginx测试

``` shell
cd /usr/local/nginx/sbin
./nginx
```

页面访问:http://localhost:8080

``` shell
#启动
cd /usr/local/nginx/sbin/
./nginx
#停止
./nginx -s stop
#重新加载
./nginx -s reload
```
