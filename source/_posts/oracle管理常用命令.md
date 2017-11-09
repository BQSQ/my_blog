---
title: oracle管理常用命令
date: 2017-11-06 20:28:30
tags: [oracle,命令]
categories: oracle
keywords: [oracle,命令,管理,常用]
---

本文总结oracle一些常用的命令查询,主要针对管理操作。

### 数据库监听
``` sql
--启动监听服务：
lsnrctl start
--停止监听服务：
lsnrctl stop
--查看监听状态：
lsnrctl status
```
<!--more -->

### 用户相关
``` sql
--创建临时表空间
create temporary tablespace TMP
tempfile '/oracle11g/product/11.2.0/oradata/orcl/TMP.dbf'
size 50m
autoextend on
next 50m maxsize 10240m
extent management local;
--创建表空间
create tablespace space
logging 
datafile '/oracle11g/product/11.2.0/oradata/orcl/space.dbf'
size 50m
autoextend on
next 50m maxsize 10240m
extent management local;

--创建用户并赋予表空间
create user username  identified by password default tablespace space  temporary tablespace TMP;

--赋予用户表空间
alter user username default tablespace space;

--赋予连接等权限
GRANT CONNECT                TO username;
GRANT EXP_FULL_DATABASE      TO username; 
GRANT IMP_FULL_DATABASE      TO username;
GRANT RESOURCE               TO username;
GRANT UNLIMITED TABLESPACE   TO username;
GRANT DEBUG CONNECT SESSION  TO username;
grant create session 		 To username;

--如果使用oracle加密解密需要单独赋予权限
GRANT EXECUTE ON DBMS_CRYPTO TO username;
grant execute on dbms_crypto To username;

--删除用户,cascade级联删除用户的关联对象
drop user username cascade;

--修改用户密码
alter user username identified by newPassword;

--查询当前用户角色
select * from user_role_privs;
select * from session_privs;

--查看当前用户的系统权限和表级权限
select * from user_sys_privs;
select * from user_tab_privs

--显示当前用户
show user;

--删除表空间
drop tablespace space_name including contents and datafiles;
--修改表空间大小（注：修改=可以增大，可以减小）
alter database datafile '/u01/app/oracle/oradata/ORCL/ittbank.dbf' resize 200m;
--增加表空间大小（注：增加=只能增大，不能减少）
alter tablespace space_name add datafile '/u01/app/oracle/oradata/ORCL/ittbank.dbf' size 2048m;
--查询数据库文件：
select * from dba_data_files;
--查询当前存在的表空间：
select * from v$tablespace;
--表空间情况
select tablespace_name,sum(bytes)/1024/1024 from dba_data_files group by tablespace_name;
--查询表空间剩余空间
select tablespace_name,sum(bytes)/1024/1024 from dba_free_space group by tablespace_name;
```

### 连接数
``` sql
--查询用户连接数
select count(*) from sys.v_$session;
--修改连接数：（注：要重启数据库）
alter system set processes=1000 scope=spfile;
--查询最大连接数
select value from v$parameter where name = 'processes';
--关闭
shutdown immediate;
--启动
startup;
```
### 锁表查询

``` sql
--锁表查询SQL
SELECT object_name, machine, s.sid, s.serial# 
FROM gv$locked_object l, dba_objects o, gv$session s 
WHERE l.object_id　= o.object_id 
AND l.session_id = s.sid;  

--释放SESSION SQL: 
alter system kill session 'sid, serial#'; 
ALTER system kill session '23, 1647'; 
```
### 查看数据库版本
``` sql
--查看数据库版本
Select * from v$version;
--查看数据库的创建日期和归档方式
Select Created, Log_Mode, Log_Mode From V$Database;
```
### 查看还没提交的事务
``` sql
select * from v$locked_object;
select * from v$transaction;
```

### 查看耗资源的进程(top session)
``` sql
select s.schemaname schema_name,decode(sign(48 - command), 1, to_char(command), 'Action Code #' ||to_char(command) ) action,status session_status,s.osuser os_user_name,s.sid,p.spid,s.serial# serial_num,nvl(s.username,'[Oracleprocess]') user_name,s.terminal terminal,s.program program,st.value criteria_value from v$sesstat st,v$session s,v$process p where st.sid = s.sid and st.statistic# = to_number('38') and ('ALL'='ALL' or s.status ='ALL') AND p.addr=s.paddr order by st.value desc,p.spid asc,s.username asc,s.osuser asc;
```

### oracle重建索引
``` sql
Declare   
    L_Sql Varchar2(32767) := '';  
Begin  
    For indexRow In   
    (  
        Select *   
        From user_indexes   
        Where tablespace_name = 'TableSpace' and status = 'VALID' And Temporary = 'N'  
    )   
    Loop  
           L_Sql := 'alter index ' || indexRow.index_name || ' rebuild ';  
           dbms_output.put_line(L_Sql);  
           EXECUTE IMMEDIATE L_Sql;           
    End Loop;  
End;
```
### 重复数据查询和删除
``` sql
select count(*)
  from t_cem_rbac_dataaccess d1
 where (d1.corpid, d1.objectid, d1.dataaccesstypeid,d1.dataaccessid,d1.accessmode,d1.is_exclude,d1.datatypeid,d1.createtime) in
       (select d2.corpid,d2.objectid,d2.dataaccesstypeid,d2.dataaccessid,d2.accessmode,d2.is_exclude,d2.datatypeid,d2.createtime
          from t_cem_rbac_dataaccess d2
         group by d2.corpid,d2.objectid,d2.dataaccesstypeid,d2.dataaccessid,d2.accessmode,d2.is_exclude,d2.datatypeid,d2.createtime
        having count(*) > 1);
```
### 保留rowid最小的那条数据
``` sql
delete from t_cem_rbac_dataaccess d1
 where (d1.corpid, d1.objectid, d1.dataaccesstypeid, d1.dataaccessid,
        d1.accessmode, d1.is_exclude, d1.datatypeid, d1.createtime) in
       (select d2.corpid,d2.objectid,d2.dataaccesstypeid,d2.dataaccessid,
               d2.accessmode,d2.is_excluded2.datatypeid,d2.createtime
          from t_cem_rbac_dataaccess d2
         group by d2.corpid,d2.objectid,d2.dataaccesstypeid,d2.dataaccessid,
                  d2.accessmode,d2.is_exclude,d2.datatypeid,d2.createtime
        having count(*) > 1)
   and rowid not in (select min(rowid)
                       from t_cem_rbac_dataaccess d3
                      group by d3.corpid,d3.objectid,d3.dataaccesstypeid,
                               d3.dataaccessid,d3.accessmode,d3.is_exclude,
                               d3.datatypeid,d3.createtime
                     having count(*) > 1) 
```
### 查看日志文件
``` sql
--查看日志文件
select member from v$logfile;
```
### 查询字符集
``` sql
select * from v$nls_parameters t where t.PARAMETER='NLS_CHARACTERSET';
```
