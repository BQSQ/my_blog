---
title: docker自动编译oracle11g
date: 2018-01-27 23:16:26
tags: [docker,oracle11g,创建]
categories: [Docker,oracle11g]
keywords: [docker,oracle11g,封装oracle11g,自动编译oracle11g]
---

本文章根据[jaspeen/oracle-11g](https://hub.docker.com/r/jaspeen/oracle-11g/)提供的方法进行自动化安装oracle并封装成images。

<!-- more -->

## 获取镜像
```
$ docker pull jaspeen/oracle-11g
```
该镜像适用于oracle11g 标准版/企业版。由于oracle许可限制，镜像不包含数据库本身，所以要首先从外部目录运行安装它。   
**注意：此镜像只作为开发学习使用。**

## 用法
首先从[官方网站下载](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index-092322.html)oracle的安装文件并把它们解压到安装目录(**install_folder**) 。
```
docker run --privileged --name oracle11g -p 1521:1521 -v <install_folder>:/install jaspeen/oracle-11g
```

等待运行安装完成后就要提交已经安装和配置好的oracle数据库。   
```
docker commit oracle11g oracle11g-installed
```

## 基本信息
* 数据库信息
```
数据库本地文件夹在 /opt/oracle
数据库用户:SYS/oracle
端口:1521
SID:orcl
```
* 系统用户
```
•root/install
•oracle/install
```
* 可以对dpdump文件夹做映射
```
docker run -d --privileged --name oracle11g -p 1521:1521 -v <install_folder>:/install -v <local_dpdump>:/opt/oracle/dpdump jaspeen/oracle-11g
```
执行impdp/expdp操作的时候可以使用如下命令:
```
docker exec -it oracle11g impdp ..
```

* 在Windows系统docker-machin下你可能会遇到如下问题   
*Exception: ORA-31640: unable to open dump file "/opt/oracle/dpdump/xxx.dmp" for read*
变通办法如下：
```
Attach to container: docker exec -it <container>

Login as oracle: su oracle

Create new directory for dumps on aufs(not mounted from external): mkdir /opt/oracle/dpdump_local

Change oracle data_pump_dir via sqlplus under SYS user: create or replace directory data_pump_dir as '/opt/oracle/dpdump_local';

Copy necessary dump from dpdump to dpdump_local directory
```
