---
title: 记录一个oracle事务的bug
date: 2017-08-06 15:49:17
tags: [oracle,事务,truncate,delete from]
categories: oracle
keywords: [oracle,myBatis,事务失效,truncate table,delete from,存储过程,自动提交事务]
---

## 背景描述
系统使用myBatis+oracle+springmvc，service事务处理使用注解@Tx进行控制，现在遇到这样一个问题，当我在一个服务使用同一个事务，先进行对一张表进行insert或者update操作后，调用存储过程对数据进行解析操作保存在另一张表。按照我理解的方式，同一个事务下，操作成功统一提交，否则回滚。当我存储过程执行失败后回滚并抛出异常，service捕捉异常后进行回滚insert/update，但是结果并不会进行回滚insert/update操作。奇怪！也就是说存储过程执行中会自动提交事务，然而并没有做commit操作，或者就是系统中@Tx有问题。

<!-- more -->

## 问题代码(部分)描述
java中service部分，虽然在同一个事务中，但是不管存储过程调用是否成功，都会提交表的更新操作。（以下说明：更新操作只的是第一步，存储过程操作指的是第二步）
``` java
@Tx
public void updateProcess(Context ctx){
    //表更新操作 param-参数
    this.sqlMap.update("test.updateProcess", param);
    //存储过程调用
    this.sqlMap.update("test.produceHandle", param);
}
```

## 解决过程(可以直接看跳到3)
### 1. 由于已经知道存储过程抛异常出来也不会回滚表更新操作，所以怀疑是架构中@Tx事务控制问题，将事务改为手动控制。   
``` java
TransactionTemplate t = ApplicationContextHelper.getBean("txTemplate");
t.execute(new TransactionCallback<Object>() {
	@Override
	public Object doInTransaction(TransactionStatus status) {
		try {
			 //表更新操作 param-参数
            this.sqlMap.update("test.updateProcess", param);
            //存储过程调用
            this.sqlMap.update("test.produceHandle", param);
		} catch (Exception e) {
			status.setRollbackOnly();
			throw e;
		}
		return null;
	}
});
```
其中TransactionTemplate事务是使用spring的TransactionTemplate，dataSource是配置的数据源。（部分代码展示只是提供一种思路）。
``` xml
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>
<bean id="txTemplate" class="org.springframework.transaction.support.TransactionTemplate">
    <property name="transactionManager" ref="txManager"></property>
</bean>
```
*结果：这样使用后依然不行，事务无效。*

### 2. 将更新操作合并到存储过程中。   
我只执行一个语句，这下应该不会有事务的问题了。就算不配置事务也没关系，我只执行一个，要么都成功要么都失败！嘿嘿
``` java
@Tx
public void updateProcess(Context ctx){
    //存储过程调用
    this.sqlMap.update("test.produceHandle", param);
}
```
*结果：依然不行，存储过程中rollback后依然不能回滚update操作，这个可是在同一个存储过程中执行的，确定中间没有进行commit。*
### 3. 手动测试存储过程，走一步查询一下表，确定是什么时候事务进行的提交。   
跟踪调试后发现在执行**TRUNCATE TABLE**语句进行删除表的时候事务进行了提交，后面的rollback其实根本无效，度娘一番：   
*TRUNCATE是一个DDL语言，和其他所有的DDL语言一样，他将被隐式提交，不能对TRUNCATE使用ROLLBACK命令。在ORACLE中，当执行DDL语句时总是需要申请一个DDL锁，以保证DDL语句执行期间，所操作对象不会被其他SESSION修改。譬如，当执行语句ALTER TABLE T时,表T将会获得一个排它的DDL锁，语句执行结束，该锁被立即释放并隐式提交。*   
解决方式：将存储过程中所有的**TRUNCATE TABLE**改成**delete from**。
*结果：完美解决问题，事务正常进行commit和rollback。并且service中@Tx也顺利进行事务控制。其实在某些领域是不允许使用TRUNCATE语句，防止数据不能进行恢复，猜想应该是前人学习到TRUNCATE语句执行效率比delete快，就滥用TRUNCATE进行删除，然而并不知道会影响到事务管理。*   
