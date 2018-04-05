---
title: Tomcat 8 解决“At least one JAR was scanned for TLDs yet contained no TLDs”问题
date: 2018-04-05 14:54:50
tags: [tomcat,TLDs]
categories: [tomcat]
keywords: [tomcat,TLDs,scanned,no TLDs]
---

在使用tomcat8启动web应用程序的时候遇到如下信息一直在等待，虽然能够启动，但是很慢,tomcat8一直在扫描TLD文件。具体TLD是什么可以百度查询，这里只说明解决办法。

``` txt
05-Apr-2018 13:53:59.619 INFO [localhost-startStop-1] org.apache.jasper.servlet.TldScanner.scanJars At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.
```

<!-- more -->

# 解决步骤

## 修改TldScanner的打印日志级别

编辑 {CATALINA-HOME}/conf/logging.properties 文件，在文件末尾添加：

``` properties
org.apache.jasper.servlet.TldScanner.level = FINE
```

修改后重启tomcat

等待Tomcat重启完成，并且相关web components都加载完成，能够正常工作。此时，在 {CATALINA-HOME}/logs/catalina.xxxx-xx-xx.log 文件中能看到类似下面的log：

``` log
05-Apr-2018 13:59:09.672 FINE [localhost-startStop-1] org.apache.jasper.servlet.TldScanner$TldScannerCallback.scan No TLD files were found in [file:/app/tomcat/webapps/invoice02/krmobile/WEB-INF/lib/druid-1.0.13.jar]. Consider adding the JAR to the tomcat.util.scan.StandardJarScanFilter.jarsToSkip property in CATALINA_BASE/conf/catalina.properties file.
05-Apr-2018 13:59:09.684 FINE [localhost-startStop-1] org.apache.jasper.servlet.TldScanner$TldScannerCallback.scan No TLD files were found in [file:/app/tomcat/webapps/invoice02/krmobile/WEB-INF/lib/jxl.jar]. Consider adding the JAR to the tomcat.util.scan.StandardJarScanFilter.jarsToSkip property in CATALINA_BASE/conf/catalina.properties file.
```

## 获取需要排除的jar包

执行如下命令,其中/app/tomcat/server/tomcat8-invoice-02/logs/ 为tomcat的log日志目录，需要根据自己的位置进行替换,主要目的是将需要排出的jar包过滤输出到skips.txt文件

``` shell
cd {CATALINA-HOME}/logs/

egrep "No TLD files were found in \[file:[^\]+\]" /app/tomcat/server/tomcat8-invoice-02/logs/catalina.out -o | egrep "[^]/]+.jar" -o | sort | uniq | sed -e 's/.jar/.jar,\\/g' > skips.txt
```

会在skips.txt 中得到类似下面的结果：

``` txt
activation-1.1.jar,\
aopalliance-1.0.jar,\
bs3-interface-client-1.0.0-SNAPSHOT.jar,\
com.jkurrent.framework.core20150913.jar,\
com.jkurrent.framework.http20160108.jar,\
com.jkurrent.framework.id20150812.jar,\
com.jkurrent.framework.sqlmap20150911.jar,\
commons-configuration-1.3.jar,\
commons-net-1.4.1.jar,\
com.springsource.org.apache.commons.fileupload-1.2.0.jar,\
druid-1.0.13.jar,\
FastInfoset-1.2.7.jar,\
fastjson-1.2.1.jar,\
gson-2.2.4.jar,\
guava-17.0.jar,\
httpcore-4.4.1.jar,\
httpmime-4.1.2.jar,\
javax.ws.rs-api-2.0.1.jar,\
jsoup-1.7.2.jar,\
jstl-api-1.2.jar,\
jxl.jar,\
kurrent-validation-1.0-SNAPSHOT.jar,\
logback-classic-0.9.29.jar,\
logback-core-0.9.29.jar,\
mybatis-3.2.2.jar,\
ojdbc-10.2.0.2.0.jar,\
pinyin4j-2.5.0.jar,\
quartz-all-1.6.0-osgi.jar,\
spring-aop-4.0.6.RELEASE.jar,\
spring-beans-4.0.6.RELEASE.jar,\
spring-context-4.0.6.RELEASE.jar,\
spring-context-support-4.0.6.RELEASE.jar,\
spring-core-4.0.6.RELEASE.jar,\
spring-expression-4.0.6.RELEASE.jar,\
spring-jdbc-4.0.6.RELEASE.jar,\
spring-orm-4.0.6.RELEASE.jar,\
spring-tx-4.0.6.RELEASE.jar,\
spring-web-4.0.6.RELEASE.jar,\
velocity-1.5.jar,\
```

## 将需要排除的jar添加到tomcat

将上面的结果放到 {CATALINA-HOME}/conf/catalina.properties 文件中的 “tomcat.util.scan.StandardJarScanFilter.jarsToSkip=” 处，保存该文件

注意：每行都以,\结尾，最后一行只有jar包名称,没有,\字符

之后删除logging.properties文件配置org.apache.jasper.servlet.TldScanner.level = FINE

总之，就是为了得到需要排除的jar包并添加到tomcat.util.scan.StandardJarScanFilter.jarsToSkip=处。