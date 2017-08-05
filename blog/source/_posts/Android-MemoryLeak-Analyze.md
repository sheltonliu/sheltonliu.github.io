---
title: Android内存泄漏分析及MAT工具使用
date: 2017-07-10 11:19:16
tags: 
	- "内存泄漏" 
	- "MAT"
categories: "Android"
---

<font size=3>
### 1.内存泄漏是什么
* 一句话概括：就是GC垃圾回收机制漏掉的垃圾对象，无法回收
* 内存泄漏过多就会造成内存溢出

### 2.什么是垃圾回收机制？
就是当对象不具备任何引用的时候，可被回收

### 3.GC ROOT Tracing 算法
![GC Root Tracing](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/10/GC_Root_Tracing.png)

* 被GC Root 引用的对象不可被回收
* 没有被GC Root Obj所持有的对象可以被回收

### 4.再谈引用
JDK1.2之后对引用进行了扩充，分为如下4种

* 强引用(Strong Reference)：类似"object obj = new object()".只要强引用还在，垃圾回收器永远不会回收被引用的对象。

* 软饮用(Soft Reference):用来描述一些还有用，但非必须的对象。在系统将要发生内存溢出异常之前，会把这些对象列入回收范围中并进行第二次回收。如果这次回收还是没有足够的内存，才会抛出内存溢出异常。

* 弱引用(Weak Reference): 它的强度比软引用更弱一些。被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否够用都会回收该引用的对象。

* 虚引用(Phantom Reference):也称为幽灵引用，或者幻影引用，它是最弱的引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用的唯一目的就是希望能在这个对象被回收时收到一个系统通知。


### 5.垃圾收集算法
* 标记-清除算法。主要有”标记“，”清除“两个阶段。首先要标记要回收的对象，然后统一清除被标记的对象。这是最基础的收集算法，后续的收集算法都是通过这种思路改进的。
	* 主要缺点：
		* 效率不高 
		* 空间问题：会产生大量不连续的内存碎片。

 		
* 复制算法
	* 将可用的内存按容量划分为等量的两块，每次只使用其中一块，当这一块内存用完了，将存活的对象复制到另一块内存中，然后将这块内存一次性清理掉，解决碎片问题
	* 代价就是将内存缩小为原来一半。还有就是如果对象存活较高的时候就要执行较多的复制，效率会降低。

* 标记-整理算法
	* 标记的过程和”标记-清理“算法一样
	* 整理是指让所有存活的对象都向一端移动，然后直接清理掉边界以外的内存。

* 分代收集算法。 java堆分为新生代和老年代
	* 新生代：复制算法
	* 老年代：标记清理，标记整理算法	
	
	 		   


### 6.可以作为GC Root引用的点是（不被回收）：
* java stack中引用的对象
* 方法区中静态引用指向的对象
* 方法去中常量引用指向的对象
* Native方法中jni引用的对象
* Thread—活着的线程

### 7.常见的内存泄漏案例：
参考[QQ空间Android内存泄漏分析心得](https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=2649796884&idx=1&sn=92b4e344060362128e4a86d6132c3736&scene=0#wechat_redirect)

* 1：单例造成的内存泄漏
	* 解决方案
		* 将该属性的引用方式改为弱引用;
		* 如果传入Context，使用ApplicationContext;
* 2： InnerClass匿名内部类
	* 在Java中，非静态内部类 和 匿名类 都会潜在的引用它们所属的外部类，但是，静态内部类却不会。如果这个非静态内部类实例做了一些耗时的操作，就会造成外围对象不会被回收，从而导致内存泄漏。
	* 解决方案
		* 将内部类变成静态内部类;
		* 如果有强引用Activity中的属性，则将该属性的引用方式改为弱引用;
		* 在业务允许的情况下，当Activity执行onDestory时，结束这些耗时任务;
* 3：Activity Context 的不正确使用
	* 在Android应用程序中通常可以使用两种Context对象：Activity和Application。当类或方法需要Context对象的时候常见的做法是使用第一个作为Context参数。这样就意味着View对象对整个Activity保持引用，因此也就保持对Activty的所有的引用。
	* 解决方案
		* 使用ApplicationContext代替ActivityContext，因为ApplicationContext会随着应用程序的存在而存在，而不依赖于activity的生命周期；
		* 对Context的引用不要超过它本身的生命周期，慎重的对Context使用“static”关键字。Context里如果有线程，一定要在onDestroy()里及时停掉。

* 4：Handler引起的内存泄漏
	* 当Handler中有延迟的的任务或是等待执行的任务队列过长，由于消息持有对Handler的引用，而Handler又持有对其外部类的潜在引用，这条引用关系会一直保持到消息得到处理，而导致了Activity无法被垃圾回收器回收，而导致了内存泄露。
	* 解决方案
		* 可以把Handler类放在单独的类文件中，或者使用静态内部类便可以避免泄露;
		* 如果想在Handler内部去调用所在的Activity,那么可以在handler内部使用弱引用的方式去指向所在Activity.使用Static + WeakReference的方式来达到断开Handler与Activity之间存在引用关系的目的。

* 5：注册监听器的泄漏
	* 解决方案
		* 使用ApplicationContext代替ActivityContext;
		* 在Activity执行onDestory时，调用反注册;

* 6：Cursor，Stream没有close，View没有recyle


* 7：集合中对象没清理造成的内存泄漏
	* 在Activity退出之前，将集合里的东西clear，然后置为null，再退出程序。

* 8： WebView造成的泄露
	* 当我们不要使用WebView对象时，应该调用它的destory()函数来销毁它，并释放其占用的内存，否则其占用的内存长期也不能被回收，从而造成内存泄露。

* 9：构造Adapter时，没有使用缓存的ConvertView	 		 		  	  		  
		 

### 8.使用AndroidStudio进行内存分析
步骤如下：

图1：
![图1](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/10/memory_tool.png)


图2：
![图2](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/10/memory_tool2.png)

### 9.通过MAT工具进行分析
第一步：如下图先导出标准的hprof文件。可以生成两个hprof文件，通过MAT比较分析。
![图1](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/10/memory_tool_hprof.png)

第二步：打开MAT工具，可以单独下载这个插件[下载](http://www.eclipse.org/mat/)
![图2](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/10/memory_tool3.png)


第三步：导入两个hprof文件，根据下图的步骤进行比较分析
![图3](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/10/memory_tool4.png)

第四步：按照下图步骤选择
![图4](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/10/memory_tool5.png)

最后：找到未释放的引用
![图5](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/10/memory_tool6.png)

参考：

[QQ空间Android内存泄漏分析心得](https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=2649796884&idx=1&sn=92b4e344060362128e4a86d6132c3736&scene=0#wechat_redirect)

《深入理解java虚拟机》










