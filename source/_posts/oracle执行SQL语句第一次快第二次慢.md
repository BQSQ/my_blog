---
title: oracle执行SQL语句第一次快第二次慢
date: 2017-07-27 15:40:55
tags: oracle
categories: oracle
keywords: [oracle,同一条sql执行时间不一样，第一次快，第二次慢]
---
## 同样的SQL语句第一次执行快于第二次   
今天在sql语句执行中遇到很奇葩的一件事情，同样一条sql语句，第一次执行需要0.526秒，第二次执行需要7.564秒，紧接着执行第三次还是7秒左右，等一会再执行又很快（1秒以内），以前重来没有遇到过，直接懵掉。按照常理来说第二次由于是一样SQL没有进行改动，会执行缓存里面的第一次执行计划，应该更加快才是，反而变得非常慢。   
自己F5查看了一下执行计划，并且将能够优化的地方也优化了一下，问题还是没有解决。既然第一次这么快，第二次怎么会慢这么多？首先排除了索引的问题，因为关联的表中都已经建立了相应的索引。其次排除了机器的问题，内存还有很多。最后定位到有可能是执行的计划问题，oracle有某些机制是我不知道的，上网查了一下，学习到了hints（提示）实现干预优化器优化这么个东西。

<!-- more -->

## sql语句案例
在查询的数据中先排序后取第一条。关联的5张表，做where条件和order by排序。

``` sql
SELECT *
  FROM (SELECT W.id, w.createtime, M.CORPID
          FROM ACT_RU_TASK RES
         INNER JOIN ACT_RU_IDENTITYLINK I
            ON I.TASK_ID_ = RES.ID_
         INNER JOIN t_fs_bill_waitspace w
            ON w.taskid = RES.ID_
         INNER JOIN t_fs_bill_maindata m
            ON w.billnumber = m.billnumber
         INNER JOIN t_fs_rbac_dataaccess da
            ON da.objectid = m.corpid
         WHERE RES.ASSIGNEE_ IS NULL
           AND I.TYPE_ = 'candidate'
           AND (I.GROUP_ID_ = 7)
           AND RES.SUSPENSION_STATE_ = 1
           AND w.checkresult = '100033'
           AND m.billtype = 'ZJ03'
           AND da.dataaccesstypeid = 7
           AND da.accessmode = 2
           AND da.dataaccessid = (select e.accountid
                                    from t_fs_sys_employee e
                                   where e.id = 10311)
        
         ORDER BY createtime)
 where rownum = 1
```
## 问题解决方案
百度之后看到hints这么个影响oracle优化器的提示。   
[hints介绍参考地址](http://czmmiao.iteye.com/blog/1478465)   
读了之后决定使用**/\*+RULE\*/** 对语句块选择基于规则的优化方法。   
*说明：现有oracle11g的优化器做的很好的，只不过有些特殊的情况需要我们指定使用哪种规则，所以不要随意使用hints。*   
**解决后的sql：**   
``` sql
SELECT /*+RULE*/ *
  FROM (SELECT W.id, w.createtime, M.CORPID
          FROM ACT_RU_TASK RES
         INNER JOIN ACT_RU_IDENTITYLINK I
            ON I.TASK_ID_ = RES.ID_
         INNER JOIN t_fs_bill_waitspace w
            ON w.taskid = RES.ID_
         INNER JOIN t_fs_bill_maindata m
            ON w.billnumber = m.billnumber
         INNER JOIN t_fs_rbac_dataaccess da
            ON da.objectid = m.corpid
         WHERE RES.ASSIGNEE_ IS NULL
           AND I.TYPE_ = 'candidate'
           AND (I.GROUP_ID_ = 7)
           AND RES.SUSPENSION_STATE_ = 1
           AND w.checkresult = '100033'
           AND m.billtype = 'ZJ03'
           AND da.dataaccesstypeid = 7
           AND da.accessmode = 2
           AND da.dataaccessid = (select e.accountid
                                    from t_fs_sys_employee e
                                   where e.id = 10311)
        
         ORDER BY createtime)
 where rownum = 1
```