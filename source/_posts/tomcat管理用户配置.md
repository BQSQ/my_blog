---
title: tomcat管理用户配置
date: 2017-07-05 15:16:59
tags: [tomcat,tomcat配置]
categories: tomcat
keywords: [tomcat,tomcat管理员配置]
---

**tomcat7配置管理员信息**   
* 打开tomcat7下的~/conf/tomcat-users.xml文件,关于用户角色、管理员的信息都在这个配置文件中。
* 在配置文件<tomcat-users>节点下添加如下xml   

``` xml
<role rolename="manager"/>  
<role rolename="manager-gui"/>  
<role rolename="manager-script" />  
<user username="admin" password="admin" roles="manager,manager-gui,manager-script"/>  
```
完成后重启即可，以上。
<!-- more -->









