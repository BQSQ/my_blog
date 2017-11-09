---
title: 'oracle加密encrypt,解密decrypt'
date: 2017-11-08 23:34:17
tags: [oracle,加密，解密]
categories: oracle
keywords: [oracle,加密,解密,encrypt,decrypt]
---

oracle数据使用加密解密，我们首先要先赋予dbms_crypto权限给用户。
``` sql
grant execute on dbms_crypto to user;
```
<!-- more -->
## 加密
``` sql
CREATE OR REPLACE FUNCTION F_ENCRYPT_DATA(NUMBER_IN IN VARCHAR2,
                                          SECRETKEY IN VARCHAR2) RETURN RAW IS
  NUMBER_IN_RAW RAW(128) := UTL_I18N.STRING_TO_RAW(NUMBER_IN, 'AL32UTF8');
  KEY_NUMBER    VARCHAR2(32) := SECRETKEY;
  KEY_RAW       RAW(128) := UTL_RAW.CAST_FROM_NUMBER(KEY_NUMBER);
  ENCRYPTED_RAW RAW(128);
BEGIN
  ENCRYPTED_RAW := DBMS_CRYPTO.ENCRYPT(SRC => NUMBER_IN_RAW,
                                       TYP => DBMS_CRYPTO.DES_CBC_PKCS5,
                                       KEY => KEY_RAW);
  RETURN ENCRYPTED_RAW;
END;
```

## 解密
``` sql
CREATE OR REPLACE FUNCTION F_DECRYPT_DATA(ENCRYPTED_RAW IN RAW,
                                          SECRETKEY     IN VARCHAR2)
  RETURN VARCHAR2 IS
  DECRYPTED_RAW RAW(128);
  KEY_NUMBER    VARCHAR2(32) := SECRETKEY;
  KEY_RAW       RAW(128) := UTL_RAW.CAST_FROM_NUMBER(KEY_NUMBER);
BEGIN
  DECRYPTED_RAW := DBMS_CRYPTO.DECRYPT(SRC => ENCRYPTED_RAW,
                                       TYP => DBMS_CRYPTO.DES_CBC_PKCS5,
                                       KEY => KEY_RAW);
  RETURN UTL_I18N.RAW_TO_CHAR(DECRYPTED_RAW, 'AL32UTF8');
END;
```