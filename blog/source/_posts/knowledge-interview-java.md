---
title: 知识点总结--java高级
date: 2017-07-18 18:08:01
categories: "知识点总结"
tags:
	- "知识点总结"
	- "java高级"
---


## 7.java高级知识点
### 7.1 IO相关

<font size=4>

1.java的IO接口

* 基于字节操作IO接口(inputstream/outputstream)
* 基于字符.........(writer/reader)
* 基于磁盘.........(file)
* 基于网络.........(socket)

2.阻塞IO的通讯模型
 
* 阻塞IO模型	
 ![阻塞IO](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/14/Android_bio.png)
 
* Bio数据在写入OutputStream或者从InputStream读取时都有可能会阻塞


3.NIO 模型

 * NIO模型
 
 ![NIO模型](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/14/android_nio.png)

 	
 * 工作原理
 * 通讯模型: 服务端和客户端各自维护一个selector对象，它能检测一个或多个channel事件

* 同步和异步是相对于应用和内核的交互方式而言的，同步 需要主动去询问，而异步的时候内核在IO事件发生的时候通知应用程序，而阻塞和非阻塞仅仅是系统在调用系统调用的时候函数的实现方式而已。[参考](http://bbym010.iteye.com/blog/2100868)  

* 如果你想吃一份宫保鸡丁盖饭： 
 * 同步阻塞：你到饭馆点餐，然后在那等着，还要一边喊：好了没啊！ 
 * 同步非阻塞：在饭馆点完餐，就去遛狗了。不过溜一会儿，就回饭馆喊一声：好了没啊！ 
 * 异步阻塞：遛狗的时候，接到饭馆电话，说饭做好了，让您亲自去拿。 
 * 异步非阻塞：饭馆打电话说，我们知道您的位置，一会给你送过来，安心遛狗就可以了。

### 7.2 多线程相关
1.volatile关键字：
volatile类型的变量
 
 * 不允许线程从主内存中将变量拷贝到自己的存储空间。表示不同线程要获取改变量都是从主内存中获取。
 * 在线程中修改该类型数据，其他线程也会刷新数据

2.synchronized关键字：

* 锁机制。当前线程通过锁拿到该变量，然后修改变量，最后释放锁。之后其他线程会更新该变量的值。 volatile只使用在字段变量上，而synchronized可使用在类，方法

3.synchronized与lock

* synchronized:悲观锁机制。采用独占锁，就是其他线程需要阻塞等待该线程释放锁

* lock：乐观锁机制。假设没有冲突去完成某件事，如果遇到冲突失败，就会不断的去重试知道成功为止。

4.线程池

* 好处：
 * 降低资源消耗
 * 提高相应速度
 * 提高线程的可管理性

### 7.3 异常相关
 * java中的检查型异常和非检查型异常有什么区别？
  * 非检查型异常包括：error, RuntimeException
 * throw 和 throws的异同
  * throw出现在函数体内，是抛出的具体的exception,需要捕获
  * throws：出现在函数头上，声明方法抛出的异常
 * java中final，finalize, finally关键字的区别
  * finalize: 垃圾回收器会调用的方法


### 7.4 注解
1.系统内置标准注解：

* Override : 表示重写方法
* Deprecated ： 表示过时
* SuppressWarnnings : 抑制编译器的警告，用于lint检查

2.元注解

* @Target    ：表示修饰目标的范围 
* @Retention ：runtime, class, source, 保留的时间长短
* @Documented:
* @Inherited : 表示可以继承

3.Android support annotations

* Nullness注解
* Resource Type 注解
* Threading 注解
* Overriding Methods 注解： @CallSuper


### 7.5 java中类加载器
* 什么是类加载器
 * ClassLoader就是动态加载class文件到内存中的

* 类加载器类型
 * BootStrap ClassLoader : 启动器的类加载器，不继承ClassLoader抽象类，底层是C++编写的。
 * Extension ClassLoader: 扩展类类加载器
 * App ClassLoader: 应用类加载器

* 双亲委托模型
![模型](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/14/android_classloader.png)
* 流程是当一个classlaoder尝试加载一个类时，它会先抛给父加载器，一层层的往上抛，抛到最上层启动类加载器。如果启动类加载器无法加载，这时会自上而下的抛给其他类加载器，直到可以加载为止


* 类加载的过程
![类加载工程](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/14/android_classloader_process.png)


### 7.6 java中堆与栈
java程序运行时的内存分配策略
 
 * 静态存储区(方法区):主要存放静态数据，全局static数据和常量
 * 栈区：方法体内的局部变量都在栈上创建
 * 堆区：通常指程序运行时直接new出来的内存,由垃圾回收器进行管理

 总结：
 
 * 堆：运行时，垃圾回收，动态分配内存，存取速度(相对于栈较慢)
 * 栈：存取速度(较快，仅次于寄存器),生命周期(栈中的数据是和生命周期进行绑定的)

 

### 7.7 java中反射
1.编译时 VS 运行时

* 编译时：将java代码编译成.class文件的过程。涉及到语法，不涉及到内存
* 运行时：就是java虚拟机执行.class文件的过程。需要用到内存的调用

2.什么是反射

* 在运行状态中，对任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；

3.Android中反射的运用

* 1.通过原始的java反射机制的方式调用资源
* 2.Activity的启动过程中Activity的对象的创建 