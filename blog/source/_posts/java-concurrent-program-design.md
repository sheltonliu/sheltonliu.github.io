---
title: java 高并发程序设计
date: 2018-02-04 12:27:20
categories: "java并发编程"
tags:
	- "java 高并发程序设计"
---
<font size=4 >

# 第三章 JDK并发包

## 重入锁
* ReentrantLock. 在jdk5.0版本中性能好于synchronized. 但是在jkd6.0之后两者性能差距并不大

* 怎么理解重入锁？这中锁是可以反复进入的，这里的反复仅仅局限与一个线程

		lock.lock();
		lock.lock();
		try{
			i++;
		}finally{
			lock.unlock();
			lock.unlock();
		}	
		
一个线程连续两次获得同一把锁是允许的。 注意在释放锁的时候，需要释放相同的次数

* 中断响应
		
		lock.lockInterruptibly();		
		
* lock(); 获得锁，如果锁已经被占用则等待
* lockInterruptibly();获得锁，但优先响应中断
* tryLock();该方法立即返回。获得锁成功返回true,失败返回false
* tryLock(long time,TimeUnit unit);在给定时间内尝试获得锁
* unlock();释放锁

* 公平锁
	* 重入锁允许设置锁的公平性
		* 在构造方法中 fair为true表示公平
				
				public ReentrantLock(boolean fair);
	* synchronized 产生的锁是非公平性的
	* 特点：不会产生饥饿现象。按时间的先后顺序保证公平，只要你排队是可以获得资源的

* 就重入锁的实现来看，主要集中在java层面。包含三要素
	* 原子状态。原子状态使用CAS操作(第四章)来存储当前锁的状态		

### 重入锁的搭档：Condition条件

condition是与重入锁相关联的。类似Object.wait() Object.notify

* await() :使当前线程等待，同时释放当前锁。当其他线程中使用signal()或者signalAll()方法时，线程会重新获得锁并继续执行。或者当线程被中断时也能跳出等待。这和Object.wait()方法类似
* awaitUninterruptibly() 和 await()基本相同。但它不会在等待过程中响应中断
* singal()唤醒一个在等待中的线程
* singalAll()唤醒所有在等待中线程

### 允许多个线程同时访问：信号量(Semaphore)
信号量是对锁的扩展。无论是内部锁synchronized还是重入锁ReentrantLock一次都只允许一个线程访问一个资源。信号量却可以指定多个线程同时访问某一个资源。

	public Semaphore(int permits)
	public Semaphore(int permits, boolean fair) //第二个参数指定是否公平
	

	public void acquire():尝试获得一个准入许可。
			若无法获得则线程等待。直到线程释放一个许可，或当前线程被中断
	
	public void acquireUninterruptibly() 与acquire类似但不响应中断
	
	public boolean tryAcquire() 尝试获得一个许可，不会进行等待 立即返回
	
	public boolean tryAcquire(long timeout, TimeUnit unit)	
	
	public void release() 用于在线程访问资源后释放许可，使其他线程可以访问



### ReadWriteLock 读写锁

JDK提供的读写分离锁

  				 | 读            | 写              
------------- | ------------- | -------------
读            |非阻塞          | 阻塞
写            |阻塞            | 阻塞

在系统中，读操作远大于写操作。则读写锁就可以发挥最大功效，提升系统性能


### 倒计时器：CountDownLatch

我的理解就是主线程可以等待其他线程都处理完之后 在往下进行


### 循环栅栏：CyclicBarrier

这个计数器可以反复使用。 我的理解比如士兵集合完毕后触发CyclicBarrier构造里面的Runnable方法， 士兵任务完成之后触发Runnable方法，


## 线程复用：线程池




# 第六章 java8与并发





