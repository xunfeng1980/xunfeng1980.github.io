---
layout: post
title:  "从 volatile 到 MESI 协议"
date:   2018-01-25 10:44:18 +0800
categories: jdk
---


##volatile介绍

​	volatile英文含义为易变的，作为Java的一个最重要的关键字，没有之一，有人会说最重要的关键字是class，233。基本上需要线程安全的地方都会有它的出现。如比较常见的AQS(AbstractQueuedSynchronizer)中的state就被声明为volatile。

​	Java语言规范（Java8）的定义：

*volatile Fields:  A field may be declared volatile, in which case the Java Memory Model ensures*

*that all threads see a consistent value for the variable .*

​	大概意思就是说，Java内存模型可以保证共享变量（volatile）的多线程可见性，如果一个线程修改了共享变量，其他线程能够读取到修改值。



## volatile的实现

​	在对volatile锁修饰变量进行写入时，Jvm模板解释器除了生成正常的汇编指令外，还会额外生成一个lock指令。lock会将当前cpu的缓存行数据回写到系统内存，此外还会使得其他cpu的该数据地址无效，其实就是MESI控制协议所起的作用，这个协议可以保证所有处理器和内存中存储的数据是一致的。

​	*MESI（Modified Exclusive Shared Or Invalid) CPU缓存一致性协议*

2017.10.08 看了Inter cpu开发手册，发现说得不清楚，先不写。

2018.1.02 调试了jvm10的模板解释器，生成的东西太多，未搞懂。

2018.1.08 找到了一份国外MESIF文档，居然说的是i7处理器，读之，卒。











