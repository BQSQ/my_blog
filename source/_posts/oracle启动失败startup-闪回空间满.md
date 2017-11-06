---
title: oracle启动失败startup--闪回空间满
date: 2017-11-06 21:49:33
tags: [oracle,闪回空间]
categories: oracle
keywords: [oracle,启动失败,闪回空间满]
---

### 登录失败 
sqlplus / as sysdb登录后：
``` sql
startup
```
失败，ORA-03113: end-of-file on communication channel   
查看日志：
```
SQL> show parameter db_recovery_file_dest_size;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
db_recovery_file_dest_size	     big integer 20G
SQL> show parameter background_dump;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
background_dump_dest		     string	 /home/oracle/app/11g/diag/rdbm
						 s/orcl/orcl/trace

```

```
${ORACLEHOME}/app/11g/diag/rdbms/orcl/orcl/trace/alert_orcl.log
```

增大闪回空间：
```
shutdown immediate
startup nomount
alter system set db_recovery_file_dest_size=20480M;

启动：
alter database open;
或者：
shutdown immediate;
startup
```
查询闪回空间的大小
```
show parameter db_recovery_file_dest_size;
```
查询闪回空间使用情况
```
SQL> select * from v$recovery_file_dest;

NAME
--------------------------------------------------------------------------------
SPACE_LIMIT SPACE_USED SPACE_RECLAIMABLE NUMBER_OF_FILES
----------- ---------- ----------------- ---------------
/home/oracle/app/11g/flash_recovery_area
 2.1475E+10 4539396608		       0	     209
```

删除30天以前的归档日志：
```
rman nocatalog
connect target /
delete archivelog all completed before 'sysdate - 30';

```


启动失败日志：
```
ARC3 started with pid=24, OS id=2958 
ARC1: Archival started
ARC2: Archival started
ARC3: Archival started
ARC0: STARTING ARCH PROCESSES COMPLETE
ARC0: Becoming the 'no FAL' ARCH
ARC0: Becoming the 'no SRL' ARCH
ARC2: Becoming the heartbeat ARCH
Errors in file /home/oracle/app/11g/diag/rdbms/orcl/orcl/trace/orcl_ora_2942.trc:
ORA-19815: WARNING: db_recovery_file_dest_size of 8589934592 bytes is 100.00% used, and has 0 remaining bytes available.
************************************************************************
You have following choices to free up space from recovery area:
1. Consider changing RMAN RETENTION POLICY. If you are using Data Guard,
   then consider changing RMAN ARCHIVELOG DELETION POLICY.
2. Back up files to tertiary device such as tape using RMAN
   BACKUP RECOVERY AREA command.
3. Add disk space and increase db_recovery_file_dest_size parameter to
   reflect the new space.
4. Delete unnecessary files using RMAN DELETE command. If an operating
   system command was used to delete files, then use RMAN CROSSCHECK and
   DELETE EXPIRED commands.
************************************************************************
Errors in file /home/oracle/app/11g/diag/rdbms/orcl/orcl/trace/orcl_ora_2942.trc:
ORA-19809: limit exceeded for recovery files
ORA-19804: cannot reclaim 18508800 bytes disk space from 8589934592 limit
ARCH: Error 19809 Creating archive log file to '/home/oracle/app/11g/flash_recovery_area/ORCL/archivelog/2017_09_18/o1_mf_1_1591_%u_.arc'
Errors in file /home/oracle/app/11g/diag/rdbms/orcl/orcl/trace/orcl_arc2_2954.trc:
ORA-19815: WARNING: db_recovery_file_dest_size of 8589934592 bytes is 100.00% used, and has 0 remaining bytes available.
```
