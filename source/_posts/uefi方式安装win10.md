---
title: uefi方式安装win10
date: 2017-06-06 09:25:03
tags: [uefi,安装win10]
categories: 系统安装
keywords: [win10,uefi,gpt,window10安装,uefi模式安装win]
---

现在的主板都支持uefi+gpt模式启动系统，和原来的BIOS+MBR模式完全不一样。安装步骤都是一样的，主要还是磁盘分区那块，该文章假设你已经会原来的系统安装，并且成功安装过win系统。
<!-- more -->
## 准备
首先你需要将主板的uefi/legacy设置成uefi only，并且已经准备好了系统安装U盘或者光盘或者PE镜像。安装方式都是一样的，在安装如下界面中按组合键：shift+F10打开命令窗口。
![win10安装](http://opvqbxg2k.bkt.clouddn.com/image/wininstall/wininstall01.png)   
## 命令GPT分区
输入如下命令进行GPT分区（**重点**）：
``` shell
diskpart 
list disk
#选择你要安装的磁盘，一般是0
select disk 0 
#清除磁盘，如果有重要文件，注意备份
clean
#转换成gpt格式
convert gpt 
#创建msr分区，大小为128m
create partition msr size=128 
#创建efi分区，大小为300m
create partition efi size=300
#创建普通的NTFS分区，大小为40G 
create partition primary size=40960
#退出diskpart
eixt 
#退出命令行
exit
```
然后将系统安装在创建的primary主分区上就可以了。
*注意：该文章主要是命令备份，不适合小白。有任何问题可以留言。*

