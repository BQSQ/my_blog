---
title: oracle自定义splitstr
date: 2017-11-08 23:19:01
tags: [oracle,splitstr]
categories: oracle
keywords: [oracle,splitstr]
---

有时候我们需要将('a,b,c,d')字符分割开，这个时候就需要oracle自定义一个splitstr函数，操作起来非常简单，执行下面的function就可以了。
<!-- more -->

``` sql
CREATE OR REPLACE FUNCTION SPLITSTR(P_STRING    IN VARCHAR2,
                                    P_DELIMITER IN VARCHAR2)
  RETURN STR_SPLIT
  PIPELINED AS
  V_LENGTH NUMBER := LENGTH(P_STRING);
  V_START  NUMBER := 1;
  V_INDEX  NUMBER;
BEGIN
  WHILE (V_START <= V_LENGTH) LOOP
    V_INDEX := INSTR(P_STRING, P_DELIMITER, V_START);
  
    IF V_INDEX = 0 THEN
      PIPE ROW(SUBSTR(P_STRING, V_START));
      V_START := V_LENGTH + 1;
    ELSE
      PIPE ROW(SUBSTR(P_STRING, V_START, V_INDEX - V_START));
      V_START := V_INDEX + 1;
    END IF;
  END LOOP;

  RETURN;
END SPLITSTR;
```
