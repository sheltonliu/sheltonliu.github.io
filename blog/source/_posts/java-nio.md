---
title: java-nio
date: 2018-01-07 11:01:11
categories: "java-NIO"
tags:
	- "java nio"
	- "nio"
---
<font size=4 >

heap  			//堆

non-heap     //非堆（jvm中堆以外的空间--method area,stack）

off-heap     //离堆，jvm之外的空间（nio中用到，用的操作系统的空间）

### NIO

* 完成高速IO，而无需编写自定义native代码

* directionBuffer    
	* Unsafe
	* 不会占用jvm中堆的空间，但是会占用操作系统空间

### Stream vs Block

* java io包 基于stream技术，按照一个字节处理

* java NIO基于Block,使用分块处理数据，比流更快捷，但是编程不易

### Channel
* 打开的连接(硬件，File, Socket,程序)
* Channel和Buffer是NIO的核心对象
* 类似stream,数据通过Channel传输
* 实现：
	* FileChannel
	* SocketChannel     //tcp
	* DatagramChannel   //udp 	

	
	
### Buffer
* 缓冲区数组
* 是容器对象的核心
* 从Channel中读取的数据进入buffer中
* NIO必须通过Buffer完成数据读写，跟踪读写过程
* 术语：
	* mark        //标记
	* capacity	 //容量
	* limit		 //限制，第一个不能读写的元素索引
	* position	 //位置，读写下一个元素的索引
	* mark <=pos<=limit<=capacity 
	* cliaring    //清除
	* flipping    //拍板
	* rewinding   //回绕











