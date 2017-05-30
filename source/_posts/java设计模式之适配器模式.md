---
title: java设计模式之适配器模式
date: 2017-05-30 22:07:30
tags: [java,适配器模式]
categories: java
keywords: [java,设计模式,适配器模式,adapter]
---

适配器模式的目的，是使用一个已经存在的类，而它的接口不符合我们的需求。我们想创建一个可以复用的类，该类可以与其他不相关的类或不可预计的类协同工作。我们想使用一些已经存在的类，但是不可能对每一个都进行子类化以匹配它们的接口。类的适配器类图结构：   
![adapter](http://opvqbxg2k.bkt.clouddn.com/images/adapter/%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F.png)   

<!-- more -->

## 两种适配器模式

### 面向类的适配器模式   
举例子说明简单易懂(关键字extends和emplements)：   
Person会说日语和英语，一个Job接口要求提供日语英语和汉语都会说，根据java设计规则中的高内聚低耦合的要求，不能随便修改Person类的属性和方法，现在加入Adapeter适配器类。   
Person   
``` java
public class Person {
	public void speakJapanese(){
		System.out.println("I can speak Japanese");
	}
	public void speakEnglish(){
		System.out.println("I can speak English");
	}
}
```
Job
``` java
public interface Job {
	public void speakJapanese();
	public void speakEnglish();
	public void speakChinese();
}
```
Adapter
``` java
public class Adapter extends Person implements Job{
	@Override
	public void speakChinese() {
		System.out.println("I can speak Chinese");
	}
}
```
*说明：Adapter继承了Person，在java中是单继承就意味着它不可能继承其他的类，这样Adapter只为这一个类Person服务，所以是类适配器模式。*

### 对象的适配器模式
还是上面的例子，用对象适配器模式实现如下：   
Adapter:
``` java
public class Adapter implements Job{
	Person person;
	public Adapter(Person person){
		this.person = person;
	}
	
	@Override
	public void speakChinese() {
		System.out.println("I can speak Chinese");
	}
	
	@Override
	public void speakJapanese() {
		person.speakJapanese();
	}

	@Override
	public void speakEnglish() {
		person.speakEnglish();
	}
}
```
*说明：将源作为构造参数传入适配器，然后执行所要求的方法，这样可以为多个源进行适配。弥补了类适配器的不足。*
## 使用场景
一般在如下情况下使用适配器模式：   
- 系统需要使用现有的类，而此类的接口不符合系统的需要。
- 想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作。这些源类不一定有很复杂的接口.
