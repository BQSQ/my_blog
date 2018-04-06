---
title: SSL管理配置-自签名证书
date: 2018-04-05 19:54:58
tags: ['SSL','配置管理','安全']
categories: SSL
keywords: [SSL,安全,配置管理,SSL配置管理]
---

SSL(Secure Sockets Layer 安全套接层),及其继任者传输层安全（Transport Layer Security，TLS）是为网络通信提供安全及数据完整性的一种安全协议。TLS与SSL在传输层对网络连接进行加密。

<!-- more -->

# 商业 CA（Certificate Authority）和私有 CA

电子商务认证授权机构（CA，Certificate Authority），也称为电子商务认证中心，是负责发放和管理数字证书的权威机构，并作为电子商务交易中受信任的第三方，承担公钥体系中公钥的合法性检验的责任。其就好比颁发身份证的公安局，当我们申请身份证的时候，我们必须要提供各种资料证明这个身份证是你的，比如你的户口本、家庭成员信息，那么 CA 也是类似，对于一些正规的商业的知名的 CA，当公司或者个人申请 SSL 证书的时候，CA 会验证申请人的信息是否可信任。

CA 是证书的签发机构，它是 PKI 的核心，CA 是负责签发证书、认证证书、管理已颁发证书的机关。它要制定政策和具体步骤来验证、识别用户身份，并对用户证书进行签名，以确保证书持有者的身份和公钥的拥有权。

CA 也拥有一个证书（内含公钥）和私钥。网上的公众用户通过验证 CA 的签字从而信任 CA，任何人都可以得到 CA 的证书（含公钥），用以验证它所签发的证书。

如果用户想得到一份属于自己的证书，他应先向 CA 提出申请。在 CA 判明申请者的身份后，便为他分配一个公钥，并且 CA 将该公钥与申请者的身份信息绑在一起，并为之签名，此后，便形成证书发给申请者。如果一个用户想鉴别另一个证书的真伪，他就用 CA 的公钥对那个证书上的签字进行验证，一旦验证通过，该证书就被认为是有效的。

下面的目前世界上最知名的8家 SSL 证书 CA 供应商。

- VeriSign
- GeoTrust
- Comodo
- Digicert
- Thawte
- GoDaddy
- Network Solutions
- GlobalSign

签发 SSL 证书的 CA 供应商之间的区别主要有：机构品牌、证书加密方式、保险额度、服务与质量、浏览器支持率等。当然，CA 机构也符合「越大越好」这个说法。

当然网上也有一些能申请免费 SSL 证书的 CA 供应商，比较知名的有：

- Let's Encrypt
- StartSSL
- COMODO PositiveSSL：90天的试用期
- CloudFlare SSL
- 腾讯云DV SSL 证书
- 阿里云DV SSL证书
- 百度云 : 有免费，不支持导出，只可用于百度云产品

上面介绍的免费 SSL 证书，要说最让人放心的当属 Let's Encrypt、腾讯云和阿里云的 DV SSL 证书，其他免费的 SSL 证书，建议大家谨慎使用，对于一些重要的网站还是建议直接购买 SSL 证书。

为了区分不同的用户级别，服务端证书可以分成 DV、OV 和 EV 证书，他们之间具体有什么区别，请参考云栖社区的一篇博客[《CA 和证书那些事》](https://yq.aliyun.com/articles/3164)，分析的很专业。

如果需要在实际公共互联网（Internet）部署的 SSL 服务器，强烈推荐到商业的知名的 CA 去申请 SSL 证书。

但是在一个企业的内部的私有网络里，比如，局域网（Intranet）内，其可能会有很多企业自己的基于 Web 的网站系统（比如工作流审批、考勤、办公等）或者消息中间件服务器或者其他企业级应用系统，这些应用系统只暴露在局域网内，不会暴露在公有网络中，但是又需要有 SSL 保护各个系统之间的通信，比如存储企业所有应用系统或者个人信息的密码的 PMP 服务器（Password Management Professional），在访问和调用其 API 的时候，就一定要基于 HTTPS，因为密码可是机密信息啊。

但是出于成本考虑或者保密性要求，我们没有必要向第三方商业 CA 申请 SSL 证书，不但花钱，而且还要提供一些杂七杂八的企业证明信息，这个时候，就可以考虑在企业内部搭建一个私有的 CA，搭建已给自己的局域网（Intranet）的一个私有 CA。

# 自签名证书

自签名证书，就是自己给自己颁发证书，换句话就是说，证书的使用者和颁发者是同一个 SSL 服务器，自签名证明一般用在开发或者测试环境，在正式的生产或者商用环境，建议大家生成证书请求，然后发送给第三方的 CA 或者自己搭建的企业内部的 CA 去签名部署。

``` shell
keytool -genkey -keyalg RSA -alias selfsigned -keystore keystore.jks -storepass 123456 -validity 360 -keysize 2048
```

按照提示一步一步填写：密码暂时填写123456

``` shell
PS D:\temp> keytool -genkey -keyalg RSA -alias selfsigned -keystore keystore.jks -storepass 123456 -validity 360 -keys
ize 2048
您的名字与姓氏是什么?
  [Unknown]:  dingyongbiao
您的组织单位名称是什么?
  [Unknown]:  default
您的组织名称是什么?
  [Unknown]:  default
您所在的城市或区域名称是什么?
  [Unknown]:  henan
您所在的省/市/自治区名称是什么?
  [Unknown]:  kaifeng
该单位的双字母国家/地区代码是什么?
  [Unknown]:  0000
CN=dingyongbiao, OU=default, O=default, L=henan, ST=kaifeng, C=0000是否正确?
  [否]:  y

输入 <selfsigned> 的密钥口令
        (如果和密钥库口令相同, 按回车):
再次输入新口令:
PS D:\temp>
```

我们可以通过下面的命令查看 keystore.jks里面生成的自签名证书的详细细节。

``` shell
keytool -list  -keystore D:/temp/keystore.jks  -v
```

# 证书签名请求（CSR）

证书签名请求（CSR，Certificate Signing Request）就是一段经过编码的字符文本，其会发送给 CA，CA 签名之后，其就变成了一个 SSL 证书，所以发送给 CA 之前，其是一个 CSR 的文件，CA 签名之后，其就变成了一张真正意义上的 SSL 证书，可以用于服务器的 SSL 配置了。

在理解证书签名请求的时候，一定要记住，你生成证明签名请求的私钥一定不要发送给 CA，因为 CA 只需要你的 CSR 文件即可，不需要你提供生成证书签名申请的私钥.

使用 OpenSSL，下面的一条命令就能生成。

``` shell
openssl req -new -newkey rsa:2048 -nodes -out servername.csr -keyout servername.key
```

使用刚才生成的jks文件再执行下面命令

``` shell
keytool -certreq -keyalg RSA -alias selfsigned -file keystore.csr -keystore keystore.jks
```

生成的证书请求文件，用文本编辑器打开，其就是类似于下面的样子。

``` txt
-----BEGIN NEW CERTIFICATE REQUEST-----
MIICzjCCAbYCAQAwWTENMAsGA1UEBhMEdGVzdDENMAsGA1UECBMEdGVzdDENMAsGA1UEBxMEdGVz
dDENMAsGA1UEChMEdGVzdDENMAsGA1UECxMEdGVzdDEMMAoGA1UEAxMDZHliMIIBIjANBgkqhkiG
9w0BAQEFAAOCAQ8AMIIBCgKCAQEAkFNKJkXwviULYbDDowGf5jRjRVFpgvs35f8VeGQQB94BBo7W
d6cM1DJpiHHMDePjS4qNWgn36rCScrJx4k+cY78TjhABd9/bYvaYbZ/v1mKHxbf0CxmO51JqmOAT
TRORD26P/JANX8x8J541rPpplgAUc00dArc+Vsm9ME4/pXsI5YcrUFM7R+eiSAsMqwlVJ8Vjr2uf
KhlL0A4i1qOJ6gtOU828nvsUnI9W72aYy+/xHNxhIdFBI5MhgQRhpzljgsKiIAVTwgJVzLoPEcU1
B0FHM6jGydxH9tD0tCPYo/JhVKJOP8bocWoJJSrNmXpSyjMFIhNNYWZZ1zi8pkjbNQIDAQABoDAw
LgYJKoZIhvcNAQkOMSEwHzAdBgNVHQ4EFgQUfxP1iiUGZlrkEnesnSpGGujfZPEwDQYJKoZIhvcN
AQELBQADggEBAGTyJORYlnWz5qUg8w5aHrkNJHNqFk8eawMBwjBc+vdC08/4RVsTSzkBLu9iO0TN
ga7b6oo9INiqq1oy03t6UtZSaZ/N7rt5CLKkfycLQpiASiV3ATFwBu3VQaVc8HyvuEql3xnu4E0T
7eHfCDuDdCigodmA85ZLSyoN/SOyYyBOgOOMIWNqXCIY/dUQuprk/GEK7ihEgj3VtZrxjUI1duFt
wJgJL2cK6RJg2aw5uJYQXW0qkixMed7wtRRxFuMyWwhPJ8pwW1zPsrUkPRCL7ZMCaoSXihVIc2hD
xtcHXGa8uSNpzot8nOBYBRND54OQ4OCC2dfC6yVKdUUgsZy1jGo=
-----END NEW CERTIFICATE REQUEST-----
```

> 需要注意的是 CSR 和私钥的长度决定了其是否能被破解的难易程度，截止 2018 年，私钥长度小于 2048 位的都可以被认为是有潜在安全风险的，因为根据现有的计算机的计算能力，小于 2048 的私钥在几个月内就能根据公钥成功碰撞私钥，如果一旦私钥被破解出来，初始化 SSL 连接其对称加密的秘钥就可能被破解者获取，从而无法保证 SSL 在通信图中的信息安全。建议在生成 CSR 的时候，使用的秘钥的长度强烈推荐为 2048 位。
