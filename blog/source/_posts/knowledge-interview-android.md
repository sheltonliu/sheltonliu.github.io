---
title: 知识点总结--android相关
date: 2017-07-18 18:08:13
categories: "知识点总结"
tags:
	- "知识点总结"
	- "Android"
---

# 1.Android基础相关知识点

<font size=4>

### 1.1：广播的使用场景

* 1：同一APP具有多进程间组件的消息通讯
* 2：不同APP之间组件的通讯

### 广播的种类
* 正常广播：sendBroadCast
* 有序广播: sendOrderBroadCast
* LocalBroadCast: 只在自身App内传播

### 实现广播
* 静态注册：注册完就一直运行  
* 动态注册：跟随Activity的生命周期

### 内部实现机制
* 1：自定义广播接收者BroadCastReceiver,重写onRecvice()
* 2:通过Binder机制向AMS(Activity Manager Service)进行注册
* 3：发送者通过Binder机制向AMS发送广播
* 4：AMS查找符合条件的广播，并放到消息循环队列
* 5: 消息循环拿到此广播，回调BroadCastReceiver中的onRecvice()方法

### LocalBroadcastManager详解
* 使用它发送的广播只在自身App内传播，不用担心隐私数据泄漏
* 其他APP无法对你的APP发送该广播，不必担心安全漏洞
* 比系统的全局广播更高效
* 高效的原因：
	* 其内部是通过Handler发送Message实现的，而系统广播是通过Binder实现的，所以更高效。同时别的应用也无法向我们的应用发送该广播
	* 内部协作主要是靠两个Map集合：mReceivers,mActions.还有一个List集合mPendingBroadcasts,这个主要是存储待接收的对象

### 1.2：WebView安全漏洞
* API16及之前的版本存在远程代码安全漏洞，该漏洞源于程序源于没有正确限制使用WebView.addJavascripInterface方法
* webview写在其他容器中时。当页面退出需要两步执行
	* 在容器中把webview销毁
	* 调用webview的ondestroy方法，避免内存泄漏
* jsbridge
* webviewClient.onPageFinished改方法有坑，当正在加载的时候进行网页跳转，该方法会调用多次。最好调用WebChromeClient.onProcessChanged方法。
* webview后台耗电. 页面退出要及时释放
* webview硬件加速(3.0开始)导致的页面渲染问题:容易出现页面加载白块或闪烁的现象。需要设置webview暂时关闭硬件加速	

#### WebView内存泄漏的问题

为什么会造成泄漏？


webview执行新的操作是在新的线程中执行的，它的生命周期与Activity的生命周期不确定。当Activity销毁的时候，Webview可能会持有Activity引用，造成泄漏。原理和匿名内部类持有外部类的引用，造成外部引用无法回收是一样的

解决方法：

* 独立进程，简单暴力，不过可能涉及到进程间通讯
* 对传入WebView中使用的Context使用弱引用，动态添加WebView，就是在布局创建个ViewGroup用来放置Webview,Activity创建时add进来，在Activity停止时remove掉

### 1.3：Binder详解
#### Linux内核的基础知识
* 进程隔离 (各个进程间互不侵犯，相互隔离。)通过虚拟地址空间来实现
* 系统调用：内核层与上层应用相互分离
* binder驱动：就是一种硬件接口，操作系统通过这个硬件接口来控制设备。

#### Binder通讯机制介绍
* 为什么使用binder?
andorid使用的Linux内核拥有非常多的跨进程通讯机制，如管道，socket
	* 性能高效：binder相当于传统的socket
	* 安全：binder机制从协议本身支持身份校验，身份校验也是android权限模型的基础

* binder通讯模型
如果把每个人比作单独的进程,人与人的联系，则：
	* binder驱动：类似于手机通讯录	.
	* serviceManager: 类似于电话基站.

![binder1](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/10/Android_binder1.png)

![binder2](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/10/Android_binder2.png)


客户端Client进程持有了server进程的代理，通过代理对象协助驱动，完成了跨进程通讯 

* 到底什么是binder?
	* 通常意义下，binder指的是一种通讯机制
	* 对于server进程来说，binder指的是binder本地对象/对于client来说，binder指的是binder的代理对象。客户端对象和服务端对象是无法直接交互的，只能通过内核层的servicemanager进行交互
	* 对于传输过程而言，binder可以进程间传递对象，自动完成代理对象和服务对象之间的转换

* binder的实例AIDL		

## 2.View相关知识点
### 2.1View的绘制机制
#### 1.View树的绘制流程
* ViewRoot根布局开始绘制
* measure->layout->draw  递归过程

measure方法介绍：

* 是一种树的递归过程，自上而下进行遍历 DecorView->ViewGroup->view
* ViewGroup.LayoutParams:宽高参数
* MeasureSpec:测量规格，32位int类型，前两位表示测量模式，后30位表示在这种模式下的大小。根据测量规格测量view的长宽高

measure--重要方法

* 1.measure
* 2.onMeasure
* 3.setMeasuredDimension()

measure方法测量总结：
	
	
* 开始于我们的父控件ViewGroup,它会通过不断的遍历子控件的measure方法，然后根据ViewGroup的MeasureSpec和子view的LayoutParams来决定子视图的MeasureSpec测量规格，通过这个MeasureSpec来获得子view的宽高，然后一层层的向下传递
* 测量的调用流程，就是树形的递归过程。每一个view视图实际的宽和高都是由父视图和它本身的LayoutParams所决定的

layout介绍：

* layout也是一层层的递归过程。其ViewGroup的onlayout()方法是一个抽象方法，自定义的View需要实现onlayout()方法

draw两个容易混淆的方法：

* 经过测量和摆放之后就需要开始绘制
* invalidate()：该方法只会执行onDraw方法
	* 当View的appearance发生改变，比如状态改变（enable，focus），背景改变，隐显改变等，这些都属于appearance范畴，都会引起invalidate操作。
	* View（非容器类）调用invalidate方法只会重绘自身，ViewGroup调用则会重绘整个View树。

* requestLayout():如果布局发生变化(方向或尺寸)在自定义View中我们需要调用该方法。该方法会触发onMeasure,onLayout，但不会触发onDraw方法
	* View执行requestLayout方法，会向上递归到顶级父View中，再执行这个顶级父View的requestLayout，所以其他View的onMeasure，onLayout也可能会被调用。
	
* invalidate和postInvalidate：invalidate方法只能用于UI线程中，在非UI线程中，可直接使用postInvalidate方法，这样就省去使用handler的烦恼

### 2.2：ListView缓存
#### RecycleBin机制
![recyclebin机制](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/11/Android_listview_recyclebin.png)


listView优化：

* convertview重用 / ViewHolder
* 三级缓存/监听滑动事件。当滑动停止的时候再加载耗时操作

## 3.Android项目构建相关知识点
 
### proguard代码混淆
工作原理：

* EntryPoint: 可以理解为一种标志，是在proguard处理的过程中，
不会被处理的类和方法
	* 对配置了EntryPoint的类和方法：如果配置了但没有用到或者是多余的类，在压缩阶段就会被丢弃，减小包体积
	* 对没有配置EntryPoint的类和方法：进行重命名

	
## 4.开源框架知识点
### 4.1：OkHttp
OkHttp的使用：

* 第一步：创建OkHttpClient对象
* 第二步：创建Request对象，内部类builder调用生成
* 第三步：创建Call对象client.newCall(request).execute()//同步请求
client.newCall(request).enqueue()//异步请求

框架的核心是拦截器的使用

httpCodec: 内部是OkIO对象，OkIO其实封装了Socket


### 4.2：retrofit
retrofit使用简介

* 1：在retrofit中通过一个接口作为http请求的api接口
* 2：创建一个retrofit实例
* 调用api接口

retrofit源码剖析--动态代理

* 1.通过method把它转换成ServiceMethod
* 2.通过serviceMethod,args获取到okhttpCall对象
* 3.把okhttpcall进一步封装并返回call对象

* 1.创建一个retrofit对象
* 2.通过Retrofit.create()方法把我们定义的接口转化成接口实例并使用接口中的方法
* 3.最终的网络请求调用的是okhttp 
 

### 4.3：Butterknife原理
Butterknife使用简介

* 一个依托java的注解机制来实现代码生成的框架。
* 编译时生成的代码来进行查找，不用担心注解的性能
* 是view的注入框架，能简化我们的代码
* 采用java的注解处理技术：注入代码，编译时解析注解并生成新的java代码



Butterknife原理

* 1.开始它会扫描java代码中所有的Butterknife注解
* 2.ButterKnifeProcessor-><className>$$ViewBinder
* 3.调用bind方法加载生成的ViewBinder类
* 4.需要注意的是添加注解的字段不能是private或static, private字段只能用反射来处理了

### 4.4：Glide图片框架：
RequestManager:图片请求管理类，实现生命周期的方法


## 5.Android异常及性能优化知识点

### 5.1: ANR异常知识点
* 什么是ANR?
	* Application Not Responding
* 造成 原因：
	* 主线程耗时操作
* 解决方法：
	* 使用Asynctask
	* 使用Thread，HandlerThread 提高优先级

### 5.2：OOM异常知识点
* 什么是OOM？
	* 当前占用的内存加上我们申请的内存资源超过了Dalvik虚拟机的最大内存限制就会抛出Out of Memory异常
* 一些混淆的概念
	* 内存溢出：就是OOM
	* 内存抖动：创建了大量的临时对象，然后又被GC回收
	* 内存泄漏：被GC Root引用无法被回收的垃圾对象，内存泄漏过多造成了内存溢出
* 解决OOM
	* 有关bitmap : 图片显示，及时释放内存，图片压缩 inBitmap属性
	* 其他方法
		* listview: convertview / lru:最近最少使用的图片机制，是一个三级缓存机制
		* 避免在onDraw方法里面执行对象的创建
		* 谨慎使用多进程

### 5.3：Bitmap知识点

1.recycle
表示在释放bitmap内存的时候，会释放有关bitmap native相关的内存


2.LRU(Least recently used)：最近最少使用的对象会被清除。

* LruCache:内部是LinkedHashMap,通过双向链表来实现。如果命中就放到链表头部，链表尾部就是不常用的缓存.提供了get/put来获取添加缓存。
* trimToSize方法：把较早的缓存移除，添加新的缓存

3.计算inSampleSize

4.缩略图

* option.inJustDecodeBounds,先设置为true会先计算图片的缩放比例，然后再设置为false,将图片放到内存中

5.三级缓存		

* 网络，本地，内存


### 5.4：UI卡顿

1:原理

* 60fps->16ms.  每隔16ms就会发出信号，触发对UI的渲染。如果每次都能渲染成功，就能达到流畅画面60fps,每秒60帧。  意味着程序的大多数操作需要在16ms内完成(1000/60与等于16)
* overdraw : 过度绘制。开启GPU：减少红色，尽量出现蓝色
	* UI布局中有大量重叠的部分

2：原因分析

* 1.人为的在UI线程做轻微的耗时操作，导致UI卡顿
* 2.布局过于复杂，无法在16ms内完成渲染
* 3.view的过度绘制
* 4.同一时间动画指定次数过多导致CPU,GPU负载过重
* 5.view频繁的出发measure,layout,导致view频繁渲染

3：UI卡顿总结

* 1.布局优化
* 2.列表及Adapter优化
* 3.背景和图片等内存分配优化
* 4.避免ANR
 	
### 5.5：冷启动优化
1.什么是冷启动

* 冷启动就是在启动应用前，系统中没有该应用的任何进程信息
* 冷启动和热启动的区别：
	* 热启动：用户使用返回键退出应用，然后又重新启动应用(进程保留在后台).    直接MainActivity不会再走Application
	* 冷启动：Application-->MainActivity


2.冷启动流程

* Application的构造方法-->attachBaseContext()-->onCreate-->Activity构造-->oncreate-->onstart-->onResume-->measure-->layout-->draw显示在界面上

3.如果对冷启动的时间进行优化

* 减少onCreate工作量
* 不要在Application进行耗时操作
* 减少层级布局/开线程



### 5.6：其他优化
1.android不用静态变量存储数据

* 静态变量数据由于进程已被杀掉而被初始化
* 使用其他数据传输方式：文件/sp/contentProvider...

2.有关Sharepreference问题

* 不能跨进程同步
* 存储sp文件过大

3.内存对象序列化
序列化：将对象的信息状态转化成可存储可传输的形式

* Serializable:容易产生大量的临时变量，频繁GC内存抖动
* Parcelable: android特有，性能高。但将数据存储到磁盘这种方式Parcelable不适用。


	
## 6.热门前沿知识点
### 6.1 android插件化

1.插件化来由：
 
 * 65536/64

2.插件化要解决的问题

* DexClassLoader可以加载jar/apk/dex，可以从SD卡中加载未安装的apk
* PathClassLoader只能加载系统中已经安装过的apk
 
* 动态加载APK 
* 资源加载
* 代码加载


### 6.2 android热更新
 1.热更新流程
 
 * 1.线上检测到严重的crash
 * 2.拉出hotfix分支，并在分支上修复问题
 * 3.jenkins构建，生成补丁包
 * 4.将补丁包部署到server端，主app获取补丁
 * 5.将hotfix分支代码合并到master分支
 
 2.热更新框架介绍
 
 * Dexposed
 * Nuwa: dex分包
 
 3.热更新原理
 
 * Android类加载机制 
 * 热修复机制：
  * dexElements
  * ClassLoader会遍历这个数组
  * 线上出现Crash,我们定位找到对应的类文件，然后进行修复，然后打包成dex文件并放入到dexElements数组最前面，让classloader优先加载我们修复好的类文件
 
 
### 6.3 android进程保活
1.进程优先级

* Foreground process
* visible process
* service process
* Background process
* Empty process

2.android 进程回收策略

* Low memory killer:通过复杂的策略对进程进行打分，分数高的进程判定为bad进程，kill掉并释放内存
* OOM_ODJ:判别进程优先级

3.进程包活方案

* 利用系统广播拉活。缺点：不可控，后台关闭自启动无法拉活
* 利用系统service拉活。 onstartcomment方法，设置start_sticky自动拉活。缺点：进程短时间内kill 5次，系统不在拉起
* 利用native进程拉活。 
	* 主要思想：利用 linux 中的 fork 机制创建 Native 进程，在 Native 进程中监控主进程的存活，当主进程挂掉后，在 Native 进程中立即对主进程进行拉活。
	* 主要原理：在 Android 中所有进程和系统组件的生命周期受 ActivityManagerService 的统一管理。而且，通过 Linux 的 fork 机制创建的进程为纯 Linux 进程，其生命周期不受 Android 的管理。

* 利用 JobScheduler 机制拉活


 
### 6.4 Lint检查
1.什么是lint检查？

* Android Lint是一个静态代码分析工具，能够对项目中潜在的bug,代码的安全性，性能，国际化进行检查

2.lint工作流程
![lint工作流程](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/13/android_lint.png) 

3.如何配置lint

* 配置lint.xml文件
* 在java代码和xml布局文件中怎么禁用lint检查。java中通过注解，xml中通过tool:ignore标签
* 自定义lint
 * 原因：满足业务需求
 * 原理：自定义Detector，实现java中的screener 	
 
 
### 6.5 kotlin
 1.kotlin到底是什么
 
 * Kotlin是一种基于jvm的编程语言
 * 是对java的一种扩展
 * kotlin 支持函数式编程
 * kotlin 类与java类能相互调用

 
 

## 7.框架原理分析 

### EventBus

EventBus.getDefault().register(this);//订阅事件
EventBus.getDefault().post(object);//发布事件
EventBus.getDefault().unregister(this);//取消订阅


EventBus中的观察者通常有四种订阅函数

* （1）onEvent
	* 使用onEvent作为订阅函数，那么该事件在哪个线程发布出来的，onEvent就会在这个线程中运行，也就是说发布事件和接收事件线程在同一个线程。
* （2）onEventMainThread
	* 无论事件在哪个线程发布出来的，始终在UI线程中执行订阅事件的操作。 
* （3）onEventBackground 
	* 无论事件在哪个线程发布出来的，始终在工作线程中执行订阅事件的操作。
* （4）onEventAsync 
	* 使用这个函数作为订阅函数，那么无论事件在哪个线程发布，都会创建新的子线程在执行onEventAsync.



这四种订阅函数都是使用onEvent开头的，它们的功能稍有不同,在介绍不同之前先介绍两个概念： 

* （1）告知观察者事件发生时通过EventBus.post函数实现，这个过程叫做事件的发布； 
* （2）观察者被告知事件发生叫做事件的接收，是通过下面的四种订阅函数实现的。


参考:  

[EventBus](http://blog.csdn.net/happy_horse/article/category/5911087)



### 怎样用通俗的语言解释REST，以及RESTful？

URL定位资源，用HTTP动词（GET,POST,DELETE,DETC）描述操作。

[知乎总结](https://www.zhihu.com/question/28557115)



### Android  HTTPS 

[csdn https](http://blog.csdn.net/yanzhenjie1003/article/details/50731272)

[阿里移动安全  Android安全开发之安全使用HTTPS ](http://www.cnblogs.com/alisecurity/p/5939336.html)


### 安卓4.4采用了新的虚拟机

从Dalvik换成了ART虚拟机。

因为android是根据linux开发的，linux大部分是用c语言写的，java不是本地语言，所以想要跨平台的话就要借助于虚拟机，

用Dalvik的时候，每次运行就会重新编译一次，这就耗费了大量的性能。而ios系统与开发的程序的语言都是一样的，不用经过这步，所以总感觉很高配置的android手机没有ios流畅。


而换成ART虚拟机后，程序会在安装的时候直接预编译到手机上，即只需编译一次，这样带来的就是效率的提高，第一次安装时间会变长，而且同样的以空间换时间为代价，安装程序之后的应用程序所占的内存将偏大，但是对于现在的手机配置来说，这些都不算什么。


### SurfaceView 介绍


SurfaceView介绍

通常情况程序的View和用户响应都是在同一个线程中处理的，这也是为什么处理长时间事件（例如访问网络）需要放到另外的线程中去（防止阻塞当前UI线程的操作和绘制）。但是在其他线程中却不能修改UI元素，例如用后台线程更新自定义View（调用View的在自定义View中的onDraw函数）是不允许的。

如果需要在另外的线程绘制界面、需要迅速的更新界面或则渲染UI界面需要较长的时间，这种情况就要使用SurfaceView了。SurfaceView中包含一个Surface对象，而Surface是可以在后台线程中绘制的。Surface属于


