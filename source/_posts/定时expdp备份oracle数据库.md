---
title: 定时expdp备份oracle数据库
date: 2017-07-12 20:06:05
tags: [oracle,expdp,数据库定时备份]
categories: oracle
keywords: [oracle,expdp,数据库定时备份]
---

在测试环境中我们需要每天备份oracle中的数据，以便错误的操作、测试或者覆盖其中有价值的数据，暂时不考虑生产环境的全量增量备份策略，本文只是简单的oracle数据库使用expdp命令设置定时任务备份数据。

<!-- more -->
## 备份shell脚本
``` shell
#!/bin/sh
#获取当前时间
BACKUPTIME=$(date +%Y%m%d)
#数据库备份DATA_PUMP_DIR目录的绝对路径
DMPDIR=/u01/app/oracle/admin/orcl/dpdump
#备份的文件名
DMPNAME=bk-$BACKUPTIME.dmp
#导出日志文件
LOGNAME=bk-$BACKUPTIME.log
#压缩后的文件
ZIPNAME=bk-$BACKUPTIME.dmp.zip

expdp 用户名/密码@实例名 directory=DATA_PUMP_DIR schemas=用户名 dumpfile=$DMPNAME logfile=$LOGNAME
cd $DMPDIR
zip -9 $ZIPNAME $DMPNAME
rm -rf $DMPDIR/$DMPNAME

#删除30天以前的文件
find ./ -mtime +30 -name "bk-*" -exec rm -rf {} \;

```
## 创建定时任务
推荐linux中使用oracle用户环境执行crontab任务。加入**. ~/.bash_profile;**是为了获取用户的环境变量，因为在测试中出现expdp命令不能使用的情况。
``` shell
crontab -e
30 1 * * * . ~/.bash_profile;  /bin/sh /home/oracle/autoBackupOracle.sh
```










