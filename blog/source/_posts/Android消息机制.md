---
title: Android消息机制
date: 2017-07-06 22:55:24
tags: 
	- "Android开发艺术探索笔记"
categories: "Android"
---

## 第十章 Android消息机制

本文是《Android开发艺术探索》一书的学习笔记

### 10.1:Android消息机制概述

* Android的消息机制主要是指Handler的运行机制。Handler需要底层的MessageQueue和Looper支撑。

* MessageQueue: 单链表数据结构，是消息存储单元，不能处理消息

* Looper: 无线循环查找是否有新消息，有的话就处理，没有就一直等待
注意Looper是运行在创建Handler所在的线程中的。

* Handler通过ThreadLocal获取每个线程的Looper来构建消息循环系统。

* 线程默认是没有Looper的。如果要使用Handler，就需要为线程创建Looper

* Android主线程也就是UI线程：ActivityThread在创建的时候就会初始化Looper.这也就是Android主线程默认可以使用Handler的原因。

* 系统为什么不允许在子线程中访问UI？ 因为Android的UI控件不是线程安全的。

* 为什么UI不采用加锁机制？
	* 1：加锁会让UI访问逻辑复杂
	* 2：会让UI访问效率降低

### 10.2:Android消息机制分析

#### 1：MessageQueue工作原理
* MessageQueue单链表结构，在插入和删除中有优势
* enqueueMessage:插入
* next : 读取并移除数据。next方法是无限循环的方法，队列中没有消息该方法会一直阻塞，有消息就返回并将该数据移除

#### 2:Looper工作原理

Handler所在线程需要Looper,没有Looper线程会报错。如何在线程中创建Looper呢？ 

	new Thread("threadname"){
		@Override
		public void run(){
			Looper.prepare(); //为当前线程创建Looper
			Handler handler = new Handler();
			Looper.loop();   //开启消息循环
		}
	}
* prepareMainLooper:给主线程创建Looper使用的
* getMainLooper:  获取主线程Looper
* quit: 直接退出Looper
* quitSafely: 把队列中已有消息处理完毕后安全退出
* 如果在子线程中创建Looper,在事情处理完后调用quit终止消息循环，否则子线程一直处于等待状态。退出Looper后线程就会立刻终止。
	
#### 3:Handler工作原理
主线程消息循环模型：
ActivityThread通过ApplicationThread和AMS进行进程间通讯。AMS以进程间通讯的方式完成ActivityThread的请求后回调ApplicationThread中的Binder方法，然后ApplicationThread向H发送消息，H将逻辑切换到ActivityThread中执行，即切换到主线程中执行。

![Android消息机制](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/06/Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6.jpeg)

![Android消息机制2](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/06/Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B62.jpg)