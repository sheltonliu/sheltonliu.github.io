---
title: NIO-ByteBuffer-DirectByteBuffer-内存映射-SocketChannel
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

* 一般来说，直接内存的访问速度（读写方面）会快于堆内存
* 在申请内存空间时，堆内存速度高于直接内存
* 直接内存适合申请次数少，访问频繁的场合。 如果内存空间需要频繁申请，则不适合直接内存


### NIO

* 完成高速IO，而无需编写自定义native代码

* HeapByteBuffer

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


![03](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/myhexo/blog/MarkdownPhotos/2018/01/07/java-nio-003.png)


![01](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/myhexo/blog/MarkdownPhotos/2018/01/07/java-nio-01.png)

![02](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/myhexo/blog/MarkdownPhotos/2018/01/07/java-nio-002.png)

![04](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/myhexo/blog/MarkdownPhotos/2018/01/07/java-nio-004.png)

![05](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/myhexo/blog/MarkdownPhotos/2018/01/07/java-nio-005.png)



### HeapByteBuffer

![06](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/07/java-nio-06.png)


### allocateDirect

* 直接字节缓冲区，off-heap(离堆，jvm之外的空间)内存

![07](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/07/java-nio-07.png)



### 小结

![08](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/07/java-nio-08.png)


![09](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/07/java-nio-09.png)

![10](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/07/java-nio-10.png)

![11](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/07/java-nio-11.png)

![12](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/07/java-nio-12.png)

![13](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/07/java-nio-13.png)

![14](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/07/java-nio-14.png)

### Memory-Mapped Files

![内存映射](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/22/nio_01.jpg)

也就是将物理文件(整个或者一部分)映射到内存，通过对内存的读写进而改变物理文件。

这里所占用的内存空间是操作系统的，非jvm

加载文件到内存的用法，如下：
![读](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/22/nio_03.jpg)

![写](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/22/nio_02.jpg)
注意：FileChannel需要close


### SocketChannels

可配置阻塞与非阻塞

![简介](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/24/nio-04.jpg)

![基本操作](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/24/nio-05.jpg)

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/24/nio-11.jpg)

![server](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/24/nio-06.jpg)

![server2](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/24/nio-10.jpg)

服务端配置非阻塞模式，通过轮询的方式接收客户端

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/24/nio-13.jpg)
客户端也是可以通过阻塞与非阻塞两种模式进行配置

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/01/24/nio-12.jpg)
服务端与客户端通过非阻塞的模式建立成功，形成通道，通道是双向的，都是可以进行读写的。



