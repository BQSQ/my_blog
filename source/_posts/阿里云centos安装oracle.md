---
title: 阿里云centos安装oracle
date: 2018-06-13 23:12:41
tags: [阿里云,centos,oracle]
categories: [oracle,阿里云]
keywords: [oracle,linux,centos,安装,阿里云]
---

阿里云linux（centos7.4）下安装oracle11g，首先需要下载oracle安装文件linux.x64_11gR2_database_1of2.zip和linux.x64_11gR2_database_1of2.zip。本机环境信息：阿里云CentOS7-64bits，2c16G，100G硬盘。

<!-- more -->

# 阿里云centos安装oracle

## 阿里云默认没有swap分区，oracle安装需要

创建swap分区是为了弥补物理内存的不足，也就是虚拟内存的概念，把硬盘的一部分划分作为虚拟内存，但这个空间不是越大越好，硬盘的速度远低于内存，设置不当反而拖慢系统的速度。

阿里云的主机默认没有swap分区，可以使用free命令查看。

1. 使用dd命令创建一个swap分区: dd if=/dev/zero of=/home/swap bs=1024 count=1048576 (count的值是：size（多少M）* 1024，我这里设置的1G虚拟内存，也就是count=1024000)

2. 格式化swap分区: mkswap /home/swap

3. 把格式化后的文件分区设置为swap分区: swapon /home/swap （关闭SWAP分区命令为：[root@localhost Desktop]#swapoff /home/swap）

4. swap分区自动挂载:vi /etc/fstab 在文件末尾加上"/home/swap swap swap default 0 0"

## 安装Oracle所需的依赖包

``` shell
yum -y install  gcc gcc-c++ make binutils compat-libstdc++-33 glibc glibc-devel libaio libaio-devel libgcc libstdc++ libstdc++-devel unixODBC unixODBC-devel sysstat ksh
```

## 创建用户和组

``` shell
groupadd -g 200 oinstall  #添加oinstall组，组的id为200
groupadd -g 201 dba       #添加dba组，组的id为201
useradd -u 440 -g oinstall -G dba oracle #添加用户oracle,并specified它的id为440.
passwd oracle             #输入oracle用户的密码
id oracle                 #查看用户id和所属组。
```

## 关闭SELINUX(阿里云缺省关闭)

``` shell
vim /etc/selinux/config   #编辑配置文件,关闭SELINUX
setenforce 0              #立即关闭SELINUX
```

## 开始安装

### 使用“su - u oracle”切换到oracle账号下

把下面两个文件上传到CentOS7-64bits服务器的/app/oracle目录下（我比较喜欢把软件安装在/app下）

linux.x64_11gR2_database_1of2.zip和linux.x64_11gR2_database_2of2.zip

``` shell
unzip linux.x64_11gR2_database_1of2.zip
unzip linux.x64_11gR2_database_2of2.zip
```

在/app/oracle目录下会出现database目录。

``` shell
vim /app/oracle/database/response/db_install.rsp
```

### 修改db_install.rsp文件

``` properties
oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v11_2_0
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=可以写本机地址
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/app/oracle/oraInventory
SELECTED_LANGUAGES=zh_CN,en
ORACLE_HOME=/app/oracle/product/11.2.0/db_1
ORACLE_BASE=/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.isCustomInstall=false
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=oinstall
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
oracle.install.db.config.starterdb.globalDBName=orcl
oracle.install.db.config.starterdb.SID=orcl
oracle.install.db.config.starterdb.characterSet=AL32UTF8
oracle.install.db.config.starterdb.memoryOption=true
oracle.install.db.config.starterdb.memoryLimit=512
oracle.install.db.config.starterdb.installExampleSchemas=false
oracle.install.db.config.starterdb.enableSecuritySettings=true
oracle.install.db.config.starterdb.password.ALL=oracle2018
oracle.install.db.config.starterdb.control=DB_CONTROL
oracle.install.db.config.starterdb.dbcontrol.enableEmailNotification=false
oracle.install.db.config.starterdb.automatedBackup.enable=false
DECLINE_SECURITY_UPDATES=true
```

### 安装Oracle

oracle账号登陆,在/app/oracle/database路径下执行开始安装

``` shell
./runInstaller -silent -responseFile /app/oracle/database/response/db_install.rsp
```

安装成功之后会出现如下：

``` txt
The following configuration scripts need to be executed as the "root" user. 
 #!/bin/sh 
 #Root scripts to run

/app/oracle/oraInventory/orainstRoot.sh
/app/oracle/product/11.2.0/db_1/root.sh
To execute the configuration scripts:
 1. Open a terminal window
 2. Log in as "root"
 3. Run the scripts
 4. Return to this window and hit "Enter" key to continue

Successfully Setup Software.
```

#### 按照提示以root身份登录CentOS7系统执行如下命令

``` shell
/app/oracle/oraInventory/orainstRoot.sh
/app/oracle/product/11.2.0/db_1/root.sh
```

#### 以oracle身份登录CentOS7系统，设置环境变量

vi ~/.bash_profile

``` shell
export PATH
export ORACLE_BASE=/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
export PATH=$PATH:$ORACLE_HOME/bin
export ORACLE_SID=orcl
export TNS_ADMIN=$ORACLE_HOME/network/admin
```

使用下面命令使环境变量生效:source ~/.bash_profile

为了使sqlplus能够访问远程oracle数据库，不但要配置“TNS_ADMIN”环境变量，还需要环境变量指向的地址（我这里是/app/oracle/product/11.2.0/db_1/network/admin/）中放入tnsnames.ora文件

下面是我tnsnames.ora的内容，其中orcl是数据库名字。

``` txt
localoracle =  
  (DESCRIPTION =  
    (ADDRESS_LIST =  
      (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521))  
    )  
    (CONNECT_DATA =  
      (SERVICE_NAME = orcl)  
    )  
  )  
```

#### 建库

确认并修改/app/oracle/database/response/dbca.rsp，其中修改的都是CREATEDATABASE操作的内容，其他的不动。

``` properties
RESPONSEFILE_VERSION = "11.2.0"
OPERATION_TYPE = "createDatabase"
[CREATEDATABASE]
GDBNAME = "orcl"
SID = "orcl"
CHARACTERSET="AL32UTF8"
```

用oracle账号调用下面的命令

``` shell
dbca -silent -responseFile /app/oracle/database/response/dbca.rsp
#执行结束后需要输入2次oracle设置的密码
```

验证Oracle安装是否成功：

su - u oracle

sqlplus "/as sysdba"

select * from tabs;

如果成功运行，表示oracle已经启来，否则需要运行“startup”命令启动oracle.

#### 添加listener.ora文件

位置：/app/oracle/product/11.2.0/db_1/network/admin

文件内容

``` txt
# copyright (c) 1997 by the Oracle Corporation
# 
# NAME
#   listener.ora
# FUNCTION
#   Network Listener startup parameter file example
# NOTES
#   This file contains all the parameters for listener.ora,
#   and could be used to configure the listener by uncommenting
#   and changing values.  Multiple listeners can be configured
#   in one listener.ora, so listener.ora parameters take the form
#   of SID_LIST_<lsnr>, where <lsnr> is the name of the listener
#   this parameter refers to.  All parameters and values are
#   case-insensitive.

# <lsnr>
#   This parameter specifies both the name of the listener, and
#   it listening address(es). Other parameters for this listener
#   us this name in place of <lsnr>.  When not specified,
#   the name for <lsnr> defaults to "LISTENER", with the default
#   address value as shown below.
#
# LISTENER =
#  (ADDRESS_LIST=
#	(ADDRESS=(PROTOCOL=tcp)(HOST=localhost)(PORT=1521))
#	(ADDRESS=(PROTOCOL=ipc)(KEY=PNPKEY)))	
LISTENER=(DESCRIPTION_LIST=(DESCRIPTION=
      (ADDRESS=(PROTOCOL=TCP)(HOST=localhost)(PORT=1521))  
  )
)

# SID_LIST_<lsnr>
#   List of services the listener knows about and can connect 
#   clients to.  There is no default.  See the Net8 Administrator's
#   Guide for more information.
#
# SID_LIST_LISTENER=
#   (SID_LIST=
#	(SID_DESC=
#			#BEQUEATH CONFIG
#          (GLOBAL_DBNAME=salesdb.mycompany)
#          (SID_NAME=sid1)			
#          (ORACLE_HOME=/private/app/oracle/product/8.0.3)
#			#PRESPAWN CONFIG
#         (PRESPAWN_MAX=20)
#	  (PRESPAWN_LIST=
#           (PRESPAWN_DESC=(PROTOCOL=tcp)(POOL_SIZE=2)(TIMEOUT=1))
#         )
#        )
#       )

SID_LIST_LISTENER=
  (SID_LIST=
      (SID_DESC=
         (GLOBAL_DBNAME=orcl)
         (SID_NAME=orcl)
         (ORACLE_HOME=/app/oracle/product/11.2.0/db_1)
        (PRESPAWN_MAX=20)
        (PRESPAWN_LIST=
          (PRESPAWN_DESC=(PROTOCOL=tcp)(POOL_SIZE=2)(TIMEOUT=1))
        )
       )
      )
	
# PASSWORDS_<lsnr>
#   Specifies a password to authenticate stopping the listener.
#   Both encrypted and plain-text values can be set.  Encrypted passwords
#   can be set and stored using lsnrctl.  
#     LSNRCTL> change_password
#       Will prompt for old and new passwords, and use encryption both
#       to match the old password and to set the new one.
#     LSNRCTL> set password
#	Will prompt for the new password, for authentication with 
#       the listener. The password must be set before running the next
#       command.
#     LSNRCTL> save_config
#       Will save the changed password to listener.ora. These last two
#       steps are not necessary if SAVE_CONFIG_ON_STOP_<lsnr> is ON.
#       See below.
#
# Default: NONE
#
# PASSWORDS_LISTENER = 20A22647832FB454      # "foobar"

# SAVE_CONFIG_ON_STOP_<lsnr>
#   Tells the listener to save configuration changes to listener.ora when
#   it shuts down.  Changed parameter values will be written to the file,
#   while preserving formatting and comments.
# Default: OFF
# Values: ON/OFF
#
# SAVE_CONFIG_ON_STOP_LISTENER = ON

# USE_PLUG_AND_PLAY_<lsnr>
#   Tells the listener to contact an Onames server and register itself
#   and its services with Onames.
# Values: ON/OFF
# Default: OFF
#
# USE_PLUG_AND_PLAY_LISTENER = ON

# LOG_FILE_<lsnr>
#   Sets the name of the listener's log file.  The .log extension
#   is added automatically.
# Default=<lsnr>
#
# LOG_FILE_LISTENER = lsnr

# LOG_DIRECTORY_<lsnr>
#   Sets the directory for the listener's log file.
# Default: <oracle_home>/network/log
#
# LOG_DIRECTORY_LISTENER = /private/app/oracle/product/8.0.3/network/log

# TRACE_LEVEL_<lsnr>
#   Specifies desired tracing level.
# Default: OFF
# Values: OFF/USER/ADMIN/SUPPORT/0-16
#
# TRACE_LEVEL_LISTENER = SUPPORT

# TRACE_FILE_<lsnr>
#   Sets the name of the listener's trace file. The .trc extension
#   is added automatically.
# Default: <lsnr>
#
# TRACE_FILE_LISTENER = lsnr

# TRACE_DIRECTORY_<lsnr>
#   Sets the directory for the listener's trace file.
# Default: <oracle_home>/network/trace
#
# TRACE_DIRECTORY_LISTENER=/private/app/oracle/product/8.0.3/network/trace
# CONNECT_TIMEOUT_<lsnr>
#   Sets the number of seconds that the listener waits to get a 
#   valid database query after it has been started.
# Default: 10
#
# CONNECT_TIMEOUT_LISTENER=10
```

使用lsnrctl start命令启动侦听器
