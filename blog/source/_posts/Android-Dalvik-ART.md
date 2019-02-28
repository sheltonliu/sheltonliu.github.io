---
title: Android虚拟机Dalvik ART
date: 2019-02-27 12:00:28
tags:
	- "Android虚拟机"

categories: "Android"		
---

<font size=3>

## Dalvik虚拟机

### Dvm与Jvm的区别

* 基于的架构不同
	* jvm基于栈，所需的指令更多，会导致速度变慢
	* Dvm基于寄存器

* 执行的字节码不同
	* java: 生成 .class文件
	* apk: .dex文件。 dex工具将.class文件的冗余信息去除掉，然后把所有的.class文件整合到.dex文件中，减少IO操作

* dvm允许在有限的内存中同时运行多个进程
* dvm由zygote进程创建和初始化
* dvm有共享机制

### Dvm运行时堆

* 运行时堆用标记-清除(Mark-Sweep)算法进行GC
* 由两个Space组成。 
	* Zygote Space:用来管理Zygote进程中加载和创建的对象,不会触发GC
	* Allocation Space: 进行对象的分配与释放，不共享，每个进程都有独立的一份
	
	

## ART虚拟机

ART虚拟机是4.4发布的，但默认还是采用DVM。 Android5.0版本中默认采用ART，Dvm退出舞台

### ART与DVM的区别

* 在运行效率上
	* dvm: 每次运行，字节码都需要通过JIT进行编译为机器码，运行效率低
	* ART: 应用在安装的时候会进行预编译(AOT),将字节码预先编译成机器码存储在本地。这样程序在每次运行时就不需要编译了，提升运行效率
		* ART存在的缺点：1.安装时间长  2.字节码预先编译成机器码占用空间变多
		* ART的改进：Android7.0版本中加入JIT,作为AOT的补充。1.安装的时候不会全部编译成机器码，减少安装时间。  2.在运行时将热点代码编译成机器码，节省空间

* 在CPU的设计上
	* dvm : 是为32位CPU设计的
	* ART： 支持64位，同时兼容32位CPU

* 在垃圾回收机制上
	* ART进行改进，更加频繁的执行并行垃圾收集

* 运行时堆空间的划分上


### ART运行时堆

![ART运行时堆](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2019/02/27/dvm-art.png)
	
其中，Zygote Space和Image Space是进程间共享的
	




	
		

