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
线程本身也是占用内存空间的，大量的线程可能会导致OOM，所以对线程需要统一的管理

![Executor框架](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/03/17/jdk_thread_01.png)

其中ThreadPoolExecutor表示一个线程池。Executors类扮演线程池工厂的角色。通过Executor可以获得一个特定功能的线程池。Executor框架提供一下方法

* newFixedThreadPool()方法。 返回一个固定线程数量的线程池，线程池中的线程数量始终不变
	* 提交一个任务
		* 线程池中有空闲线程-->立即执行
		* 线程池中无空闲线程-->放入任务队列(待线程空闲时处理任务队列的任务)
                
* newSingleThreadExecutor(): 返回只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在队列中，待线程空闲按顺序执行队列中的任务

* newCachedThreadPool(): 返回可调整线程数量的线程池。所有线程在当前任务执行完毕后，将返回线程池进行复用

* newSingleThreadScheduledExecutor():返回ScheduledExecutorService对象，线程池大小1. 主要定时执行某个任务或周期性执行某个任务

* newScheduledThreadPool(): 返回ScheduledExecutorService对象。该线程池可以指定线程数量               
                
                


# 第四章 锁的优化及注意事项

## 提高“锁”性能的几点建议：

* 减少锁持有的时间。在具体的方法中添加锁
* 减小锁粒度。典型的使用场景就是ConcurrentHashMap.  默认有16个段。幸运的话可同时接受16个线程同时插入。 注意size()会对所有段进行加锁
* 读写分离锁，来替换独占锁（在读多写少的场合使用读写锁来提升系统性能）
* 锁分离 LinkedBlockingQueue,分离的put() take() 操作。两个操作分别作用于队列的前端和尾端。 通过takeLock和putLock两把锁，实现了数据的读写分离

## java虚拟机对锁优化做出的努力

* 锁偏向。 是一种对加锁操作的优化手段。 核心思想：如果一个线程获得了锁，那就进入偏向模式。当这个线程再次请求锁时，无需在做任何同步操作，从而提高性能。
	* 如果每次都是不同的线程来请求相同的锁，这样偏向模式就会失效，因此还不如不启动偏向锁。 -XX:+UseBiasedLocking 可以开启偏向锁

* 轻量级锁。 偏向锁失败后，虚拟机不会立即挂起线程。会使用轻量级锁。它只是简单的将对象头部作为指针，指向持有锁的线程堆栈内部，来判断该线程是否持有对象锁。如果线程获得轻量级锁成功，则顺利进入临界区，否则膨胀为重量级锁

* 自旋锁
	* 锁膨胀后，虚拟机做最后的努力：自旋锁。虚拟机会让当前线程做几个空循环(自旋的含义)经过若干循环后如果可以得到锁，则进入临界区。否则在操作系统层面挂起

* 锁消除
		
		public String[] createString(){
			Vector<String> v = new Vector<String>();
			for(int i=0; i<100; i++){
				v.add(Integer.toString(i));
			}
			return v.toArray(new String[]{});
		}  		

	* 上述代码v是一个单纯的局部变量。局部变量是在线程栈上分配的，属于线程私有数据，因此不可能被其他线程访问。所以此时Vector加锁没有必要。
	* 锁消除 涉及关键技术：逃逸分析。就是观察某一个变量是否逃出了某一个作用域。上述代码虚拟机会将v内部的加锁操作去除。 如果返回的是v本身，那变量v逃出了该函数，其他线程可以访问到v. 那虚拟机就不会消除v内部的锁操作。


## ThreadLocal

ThreadLocal中的set方法，也是存放的new出来的新实例。使得每个线程都有自己的实例

注意：

* 为每个线程分配不同的对象，需要在应用层面保证。ThreadLocal只是起到了简单的容器作用
* 只要线程不退出，那对象的引用就会一直存在。如果希望及时回收对象，最好使用ThreadLocal.remove()方法移除
* ThreadLocal内部是由ThreadLocalMap来实现的。 ThreadLocalMap使用了弱引用。java虚拟机进行垃圾回收时，发现弱引用，就会立即回收。
* 为每个线程分配独立的对象，典型的案例就是：多线程下产生随机数。

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2018/04/15/ThreadLocalMap_01.jpeg)

## 无锁  CAS

* 无锁操作是一种乐观的策略。 采用的是比较交换的技术。
* cas算法：包含三个参数CAS(V,E,N) 
	* v:表示要更新的变量
	* e:表示预期值
	* n:表示新值
	* 如果 V ＝ E， 则 V ＝ N;

* AtomicInteger 等
* AtomicReference(无锁的对象引用) 
	* 注意会存在ABA问题. 就是从A修改到B，再从B修改为A。最后其他线程取到的还是A值，但是这期间是修改过的了。
* AtomicStampedReference(带有时间戳的对象引用) 来解决上面的问题
	* 记录对象修改的状态 
* AtomicIntegerFieldUpdater(让普通变量也享受原子操作)
	* 注意事项：
		* Updater只能修改它可见范围内的变量。如果变量修改为private，就是不行的
		* 声明的变量必须是volatile类型的。确保对象被正确的读取。
		* 不支持static静态字段。

* 无锁的有点：
	* 具有更好的性能
	* 避免死锁		


## 有关死锁的问题

* 死锁的表现：相关的进程不再工作，并且CPU占用为0（死锁的线程不占用CPU）
	* 可以通过jps命令得到进程id,然后使用jstack命令得到线程堆栈

* 避免死锁
	* 采用无锁CAS操作
	* 采用重入锁
		* 锁中断
		* 限时等待	  


# 第六章 java8与并发





