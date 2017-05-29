---
title: Docker初体验
date: 2017-05-22 19:35:56
tags: [Docker,Docker小白]
categories: Docker
keywords: [docker,初体验,封装环境,Docker学习,learn,小白]
---

[Docker](https://www.docker.com/)是一个开源工具，它可以让创建和管理 Linux 容器变得简单。容器就像是轻量级的虚拟机，并且可以以毫秒级的速度来启动或停止。Docker 帮助系统管理员和程序员在容器中开发应用程序，并且可以扩展到成千上万的节点。容器和 VM（虚拟机）的主要区别是，容器提供了基于进程的隔离，而虚拟机提供了资源的完全隔离。虚拟机可能需要一分钟来启动，而容器只需要一秒钟或更短。容器使用宿主操作系统的内核，而虚拟机使用独立的内核。Docker 的局限性之一是，它只能用在 64 位的操作系统上。

<!-- more -->

## 重装系统遇见Docker
重装系统抱怨开发环境重新搭建复杂，同学说起docker，一个命令就可以使用oracle或者其他的环境感觉很方便，然后接触docker，当时根本就不是daocker是干什么的，完全无知的一个表白，相信第一次听到或者用到的人和我一样的感受，完全没有一个概念性的东西在脑海里，现在我就以当初小白的身份，记录下daocker简单的使用过程。（具体docker是什么可以先查一下，我的简单理解就是**镜像+容器**）

## 文件准备
使用环境是window10 X64专业版（必须，其他Windows版本暂时不行，linux支持的特别好),并且需要开启cpu的一个虚拟选项[官方说明](https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows)原文如下：
>Docker for Windows requires 64bit Windows 10 Pro and Microsoft Hyper-V. Please see What to know before you install for a full list of prerequisites.   

Hyper-V可以在控制面板-程序卸载中查看，没有是不行的：
![Hyper-V](http://opvqbxg2k.bkt.clouddn.com/image/docker01/dockerlearn01.png)

[docker下载地址](https://www.docker.com/community-edition#/download)   
[镜像地址](https://hub.docker.com/r/wnameless/oracle-xe-11g/) 由于离线文件太大，想要的话可以留言。    

## docker安装
和普通的软件一样正常安装，不多说。   
## 使用镜像（关键）
窗口+R输入cmd进入命令界面。打开镜像地址查看使用说明。docker pull相当于下载镜像。   
![docker pull wnameless/oracle-xe-11g:14.04.4](http://opvqbxg2k.bkt.clouddn.com/image/docker01/dockerlearn02.png)    
我的是离线，直接在oracle镜像目录下执行load命令（在线下载是pull，离线load）
``` bash
docker load < oracle-image.tar
```
查看镜像：
``` bash
docker images
```
然后运行oracle镜像，映射22（openssh连接地址）端口和1521（容器中oracle封装默认的端口）端口。容器中的端口只有映射到你Windows10系统的端口上才能连接使用。
``` bash
docker run -d -p 49160:22 -p 49161:1521 wnameless/oracle-xe-11g
```  
如果你想远程连接oracle执行如下
``` bash
docker run -d -p 49160:22 -p 49161:1521 -e ORACLE_ALLOW_REMOTE=true wnameless/oracle-xe-11g
```
*说明：-e是执行参数，我一般都加上--restart=always容器启动的时候自动启动oracle，并且加上一个名称--name=oracle-master,-i打开STDIN，用于控制台交互,-t 分配 tty设备，该可以支持终端登录*   
推荐执行如下命令：
``` bash
docker run -it -d --restart=always --name=oracle-master -p 49160:22 -p 49161:1521 -e ORACLE_ALLOW_REMOTE=true wnameless/oracle-xe-11g
```
查看是否已经运行：
``` bash
docker ps
```
![docker运行](http://opvqbxg2k.bkt.clouddn.com/image/docker01/dockerlearn03.png)   
以下是该容器中oracle的基本信息：   
```
hostname: localhost
port: 49161
sid: xe
username: system
password: oracle
Password for SYS & SYSTEM
```
SSH登录：
```
ssh root@localhost -p 49160
password: admin
```
## 结束
是不是很简单呢，在ssh中登录后就可以进入里面linux的环境，oracle安装在目录/u01下面。赶紧使用下docker中的oracle吧！
