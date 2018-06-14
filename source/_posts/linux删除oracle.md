---
title: linux删除oracle
date: 2018-06-13 22:52:31
tags: [oracle,linux]
categories: oracle
keywords: [oracle,linux,centos,删除]
---

# 用oracle用户登录

备份好oracle的数据（重要）。

<!-- more -->

# 使用SQL*PLUS停止数据库和停止Listener

``` shell
sqlplus / as sysdba
shutdown [immediate]
exit

lsnrctl stop
```

# 停止HTTP服务

使用root用户登录
[root@test~ ]$ $ORACLE_HOME/Apache/Apache/bin/apachectl stop

# 删除oracle文件

使用root用户登录

``` shell
#将安装目录删除(本人安装太/app/oracle下)
rm -rf /app/oracle/
#将/usr/bin下的文件删除
rm -rf /usr/local/bin/dbhome
rm -rf /usr/local/bin/oraenv
rm -rf /usr/local/bin/coraenv
#将/etc/oratab删除
rm -rf /etc/oratab
#将/etc/oraInst.loc删除
rm -rf /etc/oraInst.loc
```

# 删除启动服务

``` shell
chkconfig --del dbora
```

# 删除用户，若要重新安装,可以不删除

``` shell
 userdel –r oracle
 groupdel oinstall
 groupdel dba
```
