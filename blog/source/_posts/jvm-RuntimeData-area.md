---
title: jvm数据分区
date: 2018-01-04 00:06:57
categories: "java虚拟机"
tags:
	- "深入理解java虚拟机"
	- "jvm"
---
<font size=4 >

# 运行时数据分区

![分区](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/myhexo/blog/MarkdownPhotos/2018/01/04/jvm-001.jpg)


![分区2](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/myhexo/blog/MarkdownPhotos/2018/01/04/jvm-002.jpg)

方法区，堆区： 线程共享

java stack,本地方法栈，程序计数器：不共享


java heap又分为年轻代和年老代。

年轻代又分为：伊甸区，幸存1区，幸存2区。所有对象都是在伊甸区创建的，通过垃圾回收之后进入幸存1或幸存2，最后进入年老代


