---
title: CentOS7中firewall防火墙详解和配置以及切换为iptables防火墙
date: 2018-01-30 23:16:21
tags: [centos7,防火墙,firewall,iptables]
categories: 防火墙
keywords: [centos7,防火墙,firewall,iptables,防火墙详解,配置,firewall-cmd]
---

centos7中默认使用firewalld防火墙配置服务，而不是使用iptables命令。

<!-- more -->

### firewall配置
*注意：以下firewalld 的操作只有重启之后才有效：service firewalld restart 重启*
1. 系统配置目录
```
/usr/lib/firewalld/services
```
目录中存放定义好的网络服务和端口参数，系统参数，不能修改。   

2. 用户配置目录
```
/etc/firewalld/
```
3. 如何自定义添加端口
用户可以通过修改配置文件的方式添加端口，也可以通过命令的方式添加端口，注意，修改的内容会在/etc/firewalld/ 目录下的配置文件中还体现。   

* 命令的方式添加端口
``` shell
firewall-cmd --permanent --add-port=9527/tcp 
```
参数介绍：
```
1、firewall-cmd：是Linux提供的操作firewall的一个工具；
2、--permanent：表示设置为持久；
3、--add-port：标识添加的端口；
```
另外，firewall中有Zone的概念，可以将具体的端口制定到具体的zone配置文件中。   
例如：添加8010端口
``` 
firewall-cmd --zone=public --permanent --add-port=8010/tcp
--zone=public：指定的zone为public；
```

```
[root@app-test zones]# more public.xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <port protocol="tcp" port="8082"/>
  <port protocol="tcp" port="8080"/>
  <port protocol="tcp" port="8081"/>
</zone>
```
如果–zone=dmz 这样设置的话，会在dmz.xml文件中新增一条。

* 修改配置文件的方式添加端口
```
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas.</description>
  <rule family="ipv4">
    <source address="122.10.70.234"/>
    <port protocol="udp" port="514"/>
    <accept/>
  </rule>
  <rule family="ipv4">
    <source address="123.60.255.14"/>
    <port protocol="tcp" port="10050-10051"/>
    <accept/>
  </rule>
 <rule family="ipv4">
    <source address="192.249.87.114"/> 放通指定ip，指定端口、协议
    <port protocol="tcp" port="80"/>
    <accept/>
  </rule>
<rule family="ipv4"> 放通任意ip访问服务器的9527端口
    <port protocol="tcp" port="9527"/>
    <accept/>
  </rule>
</zone>
```
上述的一个配置文件可以很好的看出:
```
1、添加需要的规则，开放通源ip为122.10.70.234，端口514，协议tcp；
2、开放通源ip为123.60.255.14，端口10050-10051，协议tcp；/3、开放通源ip为任意，端口9527，协议tcp；
```
### firewall常用命令

1. 重启、关闭、开启firewalld.service服务
``` shell
service firewalld restart 重启
service firewalld start 开启
service firewalld stop 关闭
```
2. 查看firewall服务状态
``` shell
systemctl status firewall 
```
3. 查看firewall的状态
``` shell
firewall-cmd --state
```
4. 查看防火墙规则
``` shell
firewall-cmd --list-all 
```
``` 
[root@app-test firewalld]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp8s0
  sources:
  services: ssh dhcpv6-client
  ports: 8082/tcp 8080/tcp 8081/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```
### CentOS切换为iptables防火墙
切换到iptables首先应该关掉默认的firewalld，然后安装iptables服务。
1. 关闭firewall
``` shell
service firewalld stop
systemctl disable firewalld.service #禁止firewall开机启动
```
2. 安装iptables防火墙
``` shell
yum install -y iptables-services #安装
```
3. 编辑iptables防火墙配置
``` shell
vi /etc/sysconfig/iptables #编辑防火墙配置文件
```
下边是一个完整的配置文件：
``` shell
Firewall configuration written by system-config-firewall

Manual customization of this file is not recommended.

*filter

:INPUT ACCEPT [0:0]

:FORWARD ACCEPT [0:0]

:OUTPUT ACCEPT [0:0]

-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

-A INPUT -p icmp -j ACCEPT

-A INPUT -i lo -j ACCEPT

-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT

-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT

-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT

-A INPUT -j REJECT --reject-with icmp-host-prohibited

-A FORWARD -j REJECT --reject-with icmp-host-prohibited

COMMIT
```
:wq #保存退出
```
service iptables start #开启
systemctl enable iptables.service #设置防火墙开机启动
```
使用命令开启INPUT服务端口
``` shell
iptables -I INPUT -p tcp --dport 8011 -j ACCEPT #开启8011端口 
/etc/rc.d/init.d/iptables save #保存配置 
/etc/rc.d/init.d/iptables restart #重启服务 
/etc/init.d/iptables status #查看端口是否已经开放
```
