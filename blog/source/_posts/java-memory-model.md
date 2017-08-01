---
title: Java内存模型
date: 2017-07-30 22:51:36
categories: "java并发编程"
tags:
	- "java内存模型"
	- "java并发编程的艺术"
---

<font size=4>

# Java内存模型的基础

本文是《java并发编程的艺术》一书的学习笔记

## 1.Java内存模型的抽象结构

1.Java线程之间的通讯由Java内存模型(JMM)控制，JMM决定一个线程对共享变量的写入何时对另一个线程可见。

2.线程之间的共享变量存储在主内存中，每个线程都有一个私有的本地内存，本地内存存储了该线程以读/写共享变量的副本。本地内存是一个抽象概念，并不真实存在。

3.Java内存模型的抽象示意图如下
![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_concurrency01.png)


如果线程A与线程B之间要通讯的话，需要经历下面两个步骤。

* 1.线程A把本地内存A中更新的共享变量刷新到主内存中去。
* 2.线程B到主内存中去读取线程A之前已更新过的共享变量。

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_concurrency02.png)


## 2.happens-before简介
1.从JDK5开始，Java使用新的JSR-133内存模型。JSR-133使用happens-before的概念来阐述操作之间的内存可见性。

2.在JMM中如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系。这里的操作既可以是在一个线程内，也可以在不同线程之间。


3.与程序员密切相关的happens-before规则如下

* 程序顺序规则:一个线程中的每个操作，happens-before于该线程中的任意后续操作。
* 监视器锁规则:对一个锁的解锁，happens-before于随后对这个锁加锁。
* volatile变量规则:对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
* 传递性:如果 A happens-before B, 且 B happens-before C,那么 A happens-before C.

### 注意
两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！ happens-before仅仅要求前一个操作(执行的结果)对后一个操作可见，且前一个操作按顺序排在第二个操作之前

happens-before与JMM的关系如下：

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_concurrency03.png)


# Volatile内存语义

## Volatile的特性

理解volatile特性的一个方法是把对volatile变量的单个读/写，看成是使用同一个锁对这个读/写做了同步。下面通过具体示例说明：

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_volatile_01.png)

假设有多个线程分别调用上面程序的3个方法，这个程序在语义上和下面的程序等价

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_volatile_02.png)

如上面的程序所示，一个volatile变量的单个读/写操作，与一个普通变量的读/写操作是使用同一个锁来同步，它们之间的执行效果相同。

volatile具有以下特性

* 可见性。 当线程A对volatile变量写入时，该变量会刷新到主内存。然后当线程B读取该volatile变量时，会在主内存中进行读取，避免在自身的本地线程读取。    对一个volatile变量的读，总是能看到(任意线程)对这个volatile变量最后的写入。
* 原子性。 对任意单个volatile变量的读/写具有原子性，但类似volatile++这种复合操作不具有原子性。
* 禁止编译器处理器指令重排序。 严格限制编译器处理器对volatile变量与普通变量的重排序，确保volatile的读/写和锁的释放/获取具有相同的内存语义。


## volatile内存语义的实现

为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。

下面是基于保守策略的JMM内存屏障插入策略。

* 在每个volatile写操作的前面插入一个StoreStore屏障
* 在每个volatile写操作的后面插入一个StoreLoad屏障
* 在每个volatile读操作的后面插入一个LoadLoad屏障
* 在每个volatile读操作的后面插入一个LoadStore屏障

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_volatile_04.png)


![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_volatile_05.png)


# 锁的内存语义

1.当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中

2.当线程获取锁时，JMM会把该线程对应的本地内存置为无效。从而使被监视器保护的临界区代码必须从主内存中读取共享变量。


# final域的内存语义

1.通过为final域增加读写重排序规则，可以为java程序员提供初始化安全保证：

只要对象是正确构造的(被构造对象的引用在构造函数中没有“逸出”)，那么不需要使用同步(指lock和volatile的使用)就可以保证任意线程都能看到这个final域在构造函数中被初始化之后的值

2.为什么final引用不能从构造函数内"逸出"？

看下示例代码：
![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_final_01.png)

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_final_02.png)


假设线程A执行writer()方法，线程B执行reader()方法。这里的操作2使得对象还未完成构造前就为线程B可见。即使这里的操作2是构造函数的最后一步，且在程序中操作2排在操作1后面，执行read()方法的线程仍然可能无法看到final域被初始化后的值，因为这里的操作1和操作2可能被重排序。




# 双重检查锁定与延迟初始化

## 双重检查锁定的由来
下面是示例代码

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_dcl_01.png)


上面的代码是有问题的，问题的根源在第7行初始化的时候。
因为初始化构造的时候会存在重排序
new DclSinglenton()初始化构造的时候可以分解如下三行伪代码：

	memory = allocate(); //1.分配对象的内存空间
    ctorInstance(memory);//2.初始化对象
    instance = memory;//3.设置instance指向刚分配的内存地址

多线程情况下2，3可能会重排序，如下：
	
	memory = allocate(); //1.分配对象的内存空间
    instance = memory;//3.设置instance指向刚分配的内存地址
    					  //注意，此时对象还没有被初始化
    ctorInstance(memory);//2.初始化对象
    
![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_dcl_02.png)

第7行如果发生重排序，另一个并发执行的线程B就有可能在第4行判断instance不为null. 线程B接下来将访问instance所引用的对象，但此时这个对象可能还没有被A线程初始化！


解决方法：
1.不允许2和3重排序
2.允许2和3重排序，但不允许其他线程”看到“这个重排序。

## 基于volatile的解决方案

	public class SafeDoubleCheckedLocking{
		private volatile static Instance instance;//声明volatile
		public static Instance getInstance(){
			if (instance == null){
				synchronized (SafeDoubleCheckedLocking.class){
					if (instance == null){
						instance = new Instance(); //没有问题
					}
				}
			}
		}
	}

声明对象的引用为volatile后，禁止伪代码中的2，3重排序

## 基于类初始化的解决方案
JVM在类的初始化阶段(即在Class被加载后，且被线程使用之前)，会执行类的初始化。在执行类的初始化期间，JVM会去获取一个锁。这个锁可以同步多个线程对同一个类的初始化。

基于这个特性，可以实现另一种线程安全的延迟初始化方案

	public class StaticInnerSingleton {
	private StaticInnerSingleton() {
    }
    public static StaticInnerSingleton getInstance() {
        return SingletonHolder.sInstance;
    }
    // 静态内部类
    private static class SingletonHolder {
        private static final StaticInnerSingleton sInstance = new StaticInnerSingleton();
    }
    }

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/08/01/android_dcl_04.png)


参考：

《java并发编程的艺术》


   
    



















































