---
title: Android崩溃优化
date: 2019-02-09 10:12:19
tags:
	- "Android" 
	- "Android优化相关"
	
categories: "Android"	
---

<font size=3>

## Android的两种崩溃

* java崩溃

	* Java 崩溃就是在 Java 代码中，出现了未捕获异常，导致程序异常退出

* Native崩溃
	* Native 崩溃又是怎么产生的呢？一般都是因为在 Native 代码中访问非法地址，也可能是地址对齐出现了问题，或者发生了程序主动 abort，这些都会产生相应的 signal 信号，导致程序异常退出。
	* Chromium 的Breakpad是目前 Native 崩溃捕获中最成熟的方案。 通过Breakpad 来获取发生 native crash 时候的系统信息和线程堆栈信息。
	
		* 通过Breakpad生成minidump_stackwalker
		* 在Andoid项目中先初始化Breadpad库， 遇到Native崩溃，会产生.dump文件
		* 通过minidump_stackwalker将.dump文件生成.txt文件。 这里面的内容是发生崩溃的系统信息及地址段信息
		* 通过ndk 中提供的addr2line将地址段信息反解成出现crash的代码地址，进而找到代码出错的位置。
		
		
		
		
		
		
		
## 第三方监控(崩溃方面)		

* 腾讯：Bugly
* 阿里：啄木鸟


	




