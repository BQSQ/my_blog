---
title: oracle索引问题
date: 2017-06-29 23:42:40
tags: oracle索引
categories: oracle
keywords: [oracle,oracle索引,重建索引]
---

在项目中遇到这样的情况：   
本地数据库需要将14万条数据按照一定的规则更新到oracle中，进行反复的更新、删除和插入操作，然后将该表使用expdp导出后使用impdp导入到测试环境，执行关联表查询非常缓慢，久久不出结果，并且内存占用非常大（3G），使用plsql中的F5执行计划没有发现问题。   
初步排除：1.SQL语句问题（没变），2.网络问题（VPN），执行其他语句无此问题。
<!-- more -->
由于当天执行数据导入导出和删改操作，初步判断出是**索引问题**。   
- 检查关联表发现关联常用字段没有建立索引，
- 以前没有出现过这样的问题，这次进行数据处理后出现，怀疑是操作过程中破坏了索引，但是不明白原因，仅仅是怀疑，留下疑问待解答。   

建立索引：
``` sql
create index index_name on table_name (column_name);
```

更新索引：
``` sql
alter index index_name rebuild;
```

批量更新索引：(user_indexes用户索引表)
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


