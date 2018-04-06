---
title: OpenSSL 快速入门
date: 2018-04-06 10:30:11
tags: [OpenSSL]
categories: SSL
keywords: [SSL,安全,快速入门,OpenSSL]
---

在查看、生成、转换、导入、导出等 SSL 证书的管理方面，OpenSSL 有着得天独厚的功能是支撑。 Apache 使用它加密 HTTPS 协议，OpenSSH 使用它加密 SSH。

<!-- more -->

# OpenSSL 简介

OpenSSL 是一个稳定的、商用等级的、功能全面的、免费开源的，专为 SSL/TLS 而生一款工具，其命令功能主要分为下面三大类。

``` txt
PS D:\temp> openssl help

Standard commands
asn1parse         ca                ciphers           cms
crl               crl2pkcs7         dgst              dhparam
dsa               dsaparam          ec                ecparam
enc               engine            errstr            exit
gendsa            genpkey           genrsa            help
list              nseq              ocsp              passwd
pkcs12            pkcs7             pkcs8             pkey
pkeyparam         pkeyutl           prime             rand
rehash            req               rsa               rsautl
s_client          s_server          s_time            sess_id
smime             speed             spkac             srp
ts                verify            version           x509

Message Digest commands (see the `dgst' command for more details)
blake2b512        blake2s256        gost              md4
md5               mdc2              rmd160            sha1
sha224            sha256            sha384            sha512

Cipher commands (see the `enc' command for more details)
aes-128-cbc       aes-128-ecb       aes-192-cbc       aes-192-ecb
aes-256-cbc       aes-256-ecb       base64            bf
bf-cbc            bf-cfb            bf-ecb            bf-ofb
camellia-128-cbc  camellia-128-ecb  camellia-192-cbc  camellia-192-ecb
camellia-256-cbc  camellia-256-ecb  cast              cast-cbc
cast5-cbc         cast5-cfb         cast5-ecb         cast5-ofb
des               des-cbc           des-cfb           des-ecb
des-ede           des-ede-cbc       des-ede-cfb       des-ede-ofb
des-ede3          des-ede3-cbc      des-ede3-cfb      des-ede3-ofb
des-ofb           des3              desx              idea
idea-cbc          idea-cfb          idea-ecb          idea-ofb
rc2               rc2-40-cbc        rc2-64-cbc        rc2-cbc
rc2-cfb           rc2-ecb           rc2-ofb           rc4
rc4-40            seed              seed-cbc          seed-cfb
seed-ecb          seed-ofb
```

## 标准命令 Standard commands

下面一些命令可能比较常用：

* ca：签名证书请求；
* genpkey：生成私钥；
* pkcs12：把私钥和公钥导出为 PKCS#12 格式；
* pkcs7：把 PKCS#7 格式的证书转换成为 pem 格式；
* genrsa：基于 RSA 算法生成的密钥对；
* s_client：类似于 telnet 或者 nc 的一个客户端测试工具，可以连接到 SSL 的服务器；
* req：生成 SSL 的证书请求。

## 信息摘要 Message Digest commands

其也提供了下面的生成信息摘要的命令函数和工具：

* blake2b512/blake2s256
* gost
* md4/md5
* mdc2
* rmd160
* sha1/sha224/sha256/sha384/sha512

如果是一个自己的小型的、临时的应用，MD4/MD5 生成的安全摘要已完全够用，但是如果是需要暴露于公网，且牵涉到一些财产生命安全相关的领域，当然肯定是选用安全级别更高的 SHA-2 生成的信息摘要。

## 密码学相关的算法 Cipher commands

penSSL 不但有和管理生成 SSL 证书相关的命令，更令人惊艳的是其还是一个密码学方面的专家工具箱，其提供了非常多的密码学相关的工具函数.

大家比较常见的就是 RC（是一种对称加密，加密的密钥流和明文一样长，同样的密钥和同样的长度能确定同一个密钥流），DES（一种将 64 比特的明文加密成 64 比特的密文的对称加密算法），AES（Advanced Encryption Standard，在密码学中又称 Rijndael 加密法，是美国联邦政府采用的一种区块加密标准），BASE64（一种把二进制加密成为 ASCII 字符），CBC（CBC 模式由 IBM 发明与 1976 年，在 CBC 模式中，每个明文块先与前一个密文块进行异或后，再进行加密）等算法。

# OpenSSL 安装

[OpenSSL下载地址](http://slproweb.com/products/Win32OpenSSL.html)

安装后添加到环境变量。

# OpenSSL 实战操作

使用 OpenSSL xxx -help 可以查看具体一条命令的用法以及它的参数。

## 基本的命令

### 创建一个私钥

通过 openssl help 命令，我们可以知道通过 genrsa 可以生成一个高强度的私钥，如果直接在 openSSL 的控制台输入 genrsa，则其会默认生成一个 2048 位的私钥。

输出到文件

``` shell
genrsa -aes128 -out D:/temp/openssldemo/fd.key 2048
```

使用 RSA 密码工具查看其私钥的具体信息

``` shell
rsa -text -in D:/temp/opensslDemo/fd.key
```

``` txt
OpenSSL> rsa -text -in D:/temp/opensslDemo/fd.key
Enter pass phrase for D:/temp/opensslDemo/fd.key:
Private-Key: (2048 bit)
modulus:
    00:b9:04:e7:38:3a:74:99:f4:5e:5f:52:ba:07:fd:
    09:af:65:1a:05:a1:31:12:85:90:8c:a1:24:d0:29:
    24:30:87:d9:e6:5c:f5:47:13:3d:e6:c2:1f:c8:ab:
    5f:08:6c:81:21:49:6a:be:ae:1b:f2:a0:f0:b4:b3:
    33:63:97:37:bc:ed:97:24:4d:2a:cb:b5:22:97:9e:
    36:49:5f:b3:66:22:77:0d:fa:67:22:54:95:60:56:
    3f:9d:6a:f6:03:cb:3d:25:63:9d:14:ff:a9:91:4b:
    9b:96:24:38:ff:74:c5:6d:f6:e4:f4:5c:dc:bc:c7:
    7e:e7:ca:35:81:a4:c5:43:9e:06:d2:e9:20:1f:6e:
    cf:f4:c0:54:82:4f:d3:e8:f2:79:0f:a5:e1:bd:36:
    a8:0e:e0:bb:ec:fc:4e:b9:dd:68:fd:b5:f8:c0:e1:
    ad:70:0a:48:75:7c:de:24:ec:a3:bb:56:f5:ab:11:
    07:a9:0e:f1:e2:9f:09:01:4b:13:b2:c0:f5:fc:45:
    96:87:7c:1e:ff:48:c9:49:ed:19:cb:e7:c5:9d:88:
    45:95:48:fc:e6:45:b5:a5:dd:14:73:57:34:e5:57:
    1e:f7:2c:a3:17:73:b0:ab:ef:48:d0:29:8f:42:c3:
    ea:02:99:b9:fa:12:35:47:bf:b5:86:65:cd:14:aa:
    d6:51
publicExponent: 65537 (0x10001)
privateExponent:
    5c:6f:4c:ad:54:d7:08:4b:84:12:8f:9c:0d:7d:a7:
    a7:0f:15:af:16:57:13:ef:d2:c5:cf:84:3a:d3:33:
    17:63:e7:c3:25:52:0d:4e:59:b0:bd:ef:6f:2b:de:
    f0:b0:74:db:12:78:d8:06:d8:43:a1:90:60:56:df:
    27:b4:56:ce:76:cc:f9:ff:eb:8f:96:51:4f:fa:65:
    18:c7:4d:33:8b:a8:7b:3d:4c:e5:63:e8:b5:16:a1:
    f5:9d:88:87:60:b4:8d:c7:74:a5:17:0d:ba:5f:51:
    bd:f3:1e:de:d1:92:09:5c:3e:0a:af:92:40:66:52:
    ca:ae:c5:88:9d:af:48:16:03:17:b5:95:86:63:69:
    52:a3:8e:59:0f:42:83:c7:60:bf:f1:22:04:23:3c:
    a7:4a:de:aa:1e:9d:1e:65:70:71:a7:a8:b2:e3:4f:
    a4:90:09:5b:c2:d3:b1:40:9c:34:d9:48:4e:64:a7:
    f9:ef:fa:c0:91:2d:d8:d1:42:e7:ca:58:cd:7e:f0:
    06:17:3c:0c:d5:6a:ab:7a:4e:2c:b8:32:4a:b3:a9:
    b5:9b:34:f4:f2:a3:1e:4a:3e:f6:62:a3:f6:6b:16:
    ef:9e:d2:ee:bd:81:57:54:89:1e:c1:7d:99:9f:16:
    28:94:27:8a:1d:c6:09:fc:c0:98:0a:4b:a8:e1:53:
    d9
prime1:
    00:ec:1a:15:7b:c9:5b:7c:43:94:58:c2:0c:77:01:
    2d:fc:c3:0d:05:03:f3:d2:ad:44:7f:4c:da:21:6b:
    05:84:e1:87:94:6c:2a:d2:a2:83:38:a0:09:15:18:
    24:cd:c8:46:73:e1:25:1e:e9:71:56:a1:f1:35:ce:
    1f:63:76:1e:a0:78:eb:ab:b1:ed:20:2e:2b:47:c6:
    af:4d:5a:1d:c9:23:4a:ed:56:2c:c4:5b:96:fe:37:
    15:7c:f2:8f:52:b1:b3:2c:65:d7:75:58:84:74:ec:
    9f:c3:52:81:99:bb:4a:d4:f9:12:a1:8f:90:2b:e1:
    58:fc:a9:96:5b:d8:31:7b:87
prime2:
    00:c8:9c:b4:93:8e:b8:f5:ea:05:c6:04:1d:ee:27:
    31:95:67:0e:78:b2:66:ef:21:c8:39:a5:8c:b8:79:
    17:5b:52:35:28:ff:91:8a:29:66:5d:59:9a:41:f5:
    2e:6f:df:69:89:90:26:3f:dc:1f:7c:41:4a:16:1c:
    c4:eb:ea:a7:2d:4f:7f:e7:82:d3:ac:7c:41:55:c8:
    91:ee:3a:9a:08:b6:ed:c3:42:4f:09:bc:dd:b8:d9:
    36:7b:85:d5:b4:50:33:a5:0c:2a:9b:fe:33:35:64:
    59:4a:93:1b:18:34:4a:d0:14:30:a2:9e:09:e0:ae:
    9b:ce:7b:9d:eb:c7:03:85:67
exponent1:
    00:93:7b:6f:b4:15:81:ca:4b:c4:9c:f5:0a:28:44:
    5a:5f:ab:cf:b4:34:55:d8:62:57:89:55:8e:64:95:
    9f:aa:f3:de:67:3e:72:39:85:3e:86:de:a1:0d:c6:
    39:27:3a:55:98:09:29:d0:f7:6d:ce:f9:f5:dc:f0:
    56:f7:20:4f:dd:59:eb:8d:22:e5:c6:d6:50:3e:d3:
    c3:a9:84:03:5b:23:6a:a8:7a:ce:18:12:46:6d:a2:
    27:10:17:cc:a3:91:51:25:08:b0:e0:22:5d:0b:54:
    cc:2f:8c:98:5c:59:7c:53:31:0b:0c:54:cb:70:3d:
    a2:02:a2:44:c4:36:76:22:7d
exponent2:
    00:bd:86:66:86:fd:10:3a:ab:73:f6:e6:4e:cc:7e:
    d4:b6:3c:1d:8c:e3:a6:a1:86:bd:dd:d0:4c:48:bf:
    85:d4:6d:ae:f1:63:b9:40:d8:e9:ef:89:46:55:c7:
    e7:ae:23:58:56:71:0b:e1:ca:f9:27:ef:9a:a9:97:
    56:67:37:51:e7:59:b3:11:aa:24:86:51:01:7c:a4:
    b7:51:64:a5:bf:53:ea:fe:4c:77:d5:50:4b:fc:65:
    a1:b6:42:f3:69:9d:57:9d:37:08:4e:45:72:65:9f:
    bf:47:d8:00:81:f6:6d:33:75:cf:98:e2:4b:9c:ab:
    f9:60:b2:a2:b8:6c:55:24:43
coefficient:
    2a:d3:19:34:ec:b3:26:7f:32:bc:19:ab:96:ec:63:
    c7:1f:b6:ac:32:38:2e:f6:96:13:c5:0d:8a:73:b8:
    b8:97:7c:c7:ab:40:a4:40:0c:02:c4:b9:1a:95:c2:
    b0:b5:90:ad:19:a0:b7:c6:ca:f2:87:39:ed:30:f0:
    ae:cc:a8:43:31:52:c2:cc:b3:4b:4b:c3:ff:bc:94:
    55:ee:77:a6:e8:61:52:c3:35:02:4b:48:2c:0e:4a:
    bc:2d:40:bd:5f:2f:af:48:48:09:fb:ec:28:c7:f6:
    20:97:2b:a6:02:c3:55:6d:a7:2f:61:a3:78:9c:fc:
    1d:07:fe:ce:0c:45:f2:f0
writing RSA key
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAuQTnODp0mfReX1K6B/0Jr2UaBaExEoWQjKEk0CkkMIfZ5lz1
RxM95sIfyKtfCGyBIUlqvq4b8qDwtLMzY5c3vO2XJE0qy7Uil542SV+zZiJ3Dfpn
IlSVYFY/nWr2A8s9JWOdFP+pkUubliQ4/3TFbfbk9FzcvMd+58o1gaTFQ54G0ukg
H27P9MBUgk/T6PJ5D6XhvTaoDuC77PxOud1o/bX4wOGtcApIdXzeJOyju1b1qxEH
qQ7x4p8JAUsTssD1/EWWh3we/0jJSe0Zy+fFnYhFlUj85kW1pd0Uc1c05Vce9yyj
F3Owq+9I0CmPQsPqApm5+hI1R7+1hmXNFKrWUQIDAQABAoIBAFxvTK1U1whLhBKP
nA19p6cPFa8WVxPv0sXPhDrTMxdj58MlUg1OWbC9728r3vCwdNsSeNgG2EOhkGBW
3ye0Vs52zPn/64+WUU/6ZRjHTTOLqHs9TOVj6LUWofWdiIdgtI3HdKUXDbpfUb3z
Ht7RkglcPgqvkkBmUsquxYidr0gWAxe1lYZjaVKjjlkPQoPHYL/xIgQjPKdK3qoe
nR5lcHGnqLLjT6SQCVvC07FAnDTZSE5kp/nv+sCRLdjRQufKWM1+8AYXPAzVaqt6
Tiy4MkqzqbWbNPTyox5KPvZio/ZrFu+e0u69gVdUiR7BfZmfFiiUJ4odxgn8wJgK
S6jhU9kCgYEA7BoVe8lbfEOUWMIMdwEt/MMNBQPz0q1Ef0zaIWsFhOGHlGwq0qKD
OKAJFRgkzchGc+ElHulxVqHxNc4fY3YeoHjrq7HtIC4rR8avTVodySNK7VYsxFuW
/jcVfPKPUrGzLGXXdViEdOyfw1KBmbtK1PkSoY+QK+FY/KmWW9gxe4cCgYEAyJy0
k4649eoFxgQd7icxlWcOeLJm7yHIOaWMuHkXW1I1KP+RiilmXVmaQfUub99piZAm
P9wffEFKFhzE6+qnLU9/54LTrHxBVciR7jqaCLbtw0JPCbzduNk2e4XVtFAzpQwq
m/4zNWRZSpMbGDRK0BQwop4J4K6bznud68cDhWcCgYEAk3tvtBWBykvEnPUKKERa
X6vPtDRV2GJXiVWOZJWfqvPeZz5yOYU+ht6hDcY5JzpVmAkp0Pdtzvn13PBW9yBP
3VnrjSLlxtZQPtPDqYQDWyNqqHrOGBJGbaInEBfMo5FRJQiw4CJdC1TML4yYXFl8
UzELDFTLcD2iAqJExDZ2In0CgYEAvYZmhv0QOqtz9uZOzH7UtjwdjOOmoYa93dBM
SL+F1G2u8WO5QNjp74lGVcfnriNYVnEL4cr5J++aqZdWZzdR51mzEaokhlEBfKS3
UWSlv1Pq/kx31VBL/GWhtkLzaZ1XnTcITkVyZZ+/R9gAgfZtM3XPmOJLnKv5YLKi
uGxVJEMCgYAq0xk07LMmfzK8GauW7GPHH7asMjgu9pYTxQ2Kc7i4l3zHq0CkQAwC
xLkalcKwtZCtGaC3xsryhzntMPCuzKhDMVLCzLNLS8P/vJRV7nem6GFSwzUCS0gs
Dkq8LUC9Xy+vSEgJ++wox/YglyumAsNVbacvYaN4nPwdB/7ODEXy8A==
-----END RSA PRIVATE KEY-----
```

### 创建一个证书请求

OpenSSL 就有一个 req 的命令，专门用来创建证书请求。需要注意的是，在创建一个 SSL 的证书请求前，先要创建一个私钥来，我们可以直接使用上面的私钥，通过 -key 的参数 D:/temp/opensslDemo/fd.key 来指定，然后通过 -out 的参数指定其输出的文件路径。

``` shell
req -new -key D:/temp/openSSLDemo/fd.key -out D:/temp/openSSLDemo/fd.csr
```

需要输入你的包含私钥fd.key 的密码,然后输入证书的基本信息，比如国家、地区、城市、组织、通用名、邮箱地址等，填写提示信息后就生成了csr签名文件。

证书请求生成后，可以通过下面的命令查看生成的证书请求。

``` shell
req -text -in D:/temp/openSSLDemo/fd.csr -noout
```

其中，-text 表示已文本的方式查看，-in 后面需要知道待查看的证书的请求的路径，-noout 表示不输出其已经被编码的证书文本本身，下面为命令执行结果的输出。

### 自签名证书请求

如果你只是安装一个 SSL/TLS 服务器，比如在 IIS 或者 Tomcat 上面部署一个 Web 站点，而且这个站点只是供你自己开发测试使用，这个时候，就没有必要把证书请求发送给第三方的权威的商业 CA 去签名我们的证书请求，最快最方便的方式就是自己签署自己，从而生成一个自签名的 SSL 证书。OpenSSL 已经为我们考虑好了，其命令如下：(执行文件夹D:/temp/openSSLDemo/)

``` shell
x509 -req -days 365 -in fd.csr -signkey fd.key -out fd.crt
```

证书生成后，我们需要测试或者查看证书，这个时候，就可以使用下面的类似命令了。

``` shell
x509 -text -in fd.crt -noout
```
