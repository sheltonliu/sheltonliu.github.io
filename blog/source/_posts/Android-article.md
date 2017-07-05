---
title: Android开发艺术探索学习笔记
date: 2017-07-05 23:45:32
tags: "学习笔记"
categories: "Android"
---

## 第一章：Activity的启动模式
<font size=3>

* standard: 标准模式，每次启动Activity都会重新创建新实例。
如果使用ApplicationContext启动，则需要添加Flag_activity_new_task,创建一个新的任务栈

* singleTop  模式：  栈顶复用。如果已在栈顶则Activity 不会被重建，同时onNewIntent方法会被回调

* singleTask 模式： 栈内复用。如果在栈中存在，那么多次启动Activity都不会重新建立实例，同时onNewIntent方法会被回调
如果D所在任务栈为S1，并且当前任务栈S1的情况为ADBC,根据栈内复用原则D不会被重新创建，系统会把D切换到栈顶并调用onNewIntent方法。同时由于SingleTask默认具有clearTop效果，会导致栈内D以上的Activity全部出栈，于是最终S1的情况为AD


* singleInstance: 单实例模式  是一种加强的singleTask模式。singleTask模式的特性它都有，此外具有这种模式的Activity只能单独地位于一个任务栈中





## 第二章：Android IPC简介

### 2.2.1: 通过ps命令 查看一个包名中当前的进程信息


adb shell ps `|` grep com.ryg.chapter_2. 
其中com.ryg.chapter_2 是包名

进程名
“:remote”中“:”表示要在当前进程名前面附加上当前的包名，这是一种简单的写法。 第二此类表示方式描述的进程属于当前应用的私有进程，其他应用的组件不可以和它跑在同一个进程中

进程名不已”:”开头的进程属于全局进程，其他应用通过shareUID方式可以和它跑在同一进程中


多进程带来的问题：
所有运行在不同进程中的四大组件，只要他们之间需要通过内存来共享数据，都会共享失败，这也是多进程所带来的主要影响。

	1. 静态成员和单例模式完全失效。
	2. 线程同步机制完全失效
	3. SharedPreferences可靠性失效
	4. Application会多次创建


### 2.3 IPC基础概念介绍
Serializable:  serialVersionUID  开销大

Parcelable : 适用于android平台，组要作用与内存序列化上

避免客户端的UI线程去访问远程方法

RemoteCallbackList 是系统专门提供的用于删除跨进程listener的接口

权限验证功能：默认情况下远程服务任何人都可以连接，但这应该不是我们愿意看到的，必须要给权限加上验证功能
	
	第一种方法：我们在onBind中进行验证
	第二种方法：我们可以在服务端的onTransact方法中进行


####IPC方式的优缺点和适用场景
AIDL: Android Interface definition language   

|   | 优点  | 缺点  | 试用场景 |
|:--------------|:------------- |:---------------:| -------------:|
| Bundle        | 简单易用       |  只传输bundle支持的类型  | 四大组建通讯 |
| 文件共享        | 简单易用      |  不适合高并发,无法进程间即时通讯  | 无并发访问实时性不高的场景|
| AIDL |支持一对多并发,支持实时通讯 |使用稍复杂,需要处理好线程同步 | 一对多通讯且有RPC需求|
| Message |   支持一对多串行 , 支持实时通讯 | 不能很好处理高并发情形，不支持RPC | 低并发无RPC需求|
| ContentProvider |数据源访问强大，支持一对多并发共享  | 受约束的AIDL主要提供数据源的CRUD操作 |一对多的进程间的数据共享  |
| socket |通过字节流传输，支持一对多并发实时通讯  |细节繁琐，不支持直接的RPC | 网络数据交换 |




## 第三章：view的事件体系

3.2：view的滑动：
第一种：通过view本身提供的scrollTo/scrollBy方法来滑动
这个只能将view的内容进行移动，并不能将view本身进行移动

第二种：通过动画给view施加平移效果来实现滑动
如果希望动画后的状态得以保留还必须将fillAfter属性设置为true,否则动画完成后其动画结果会消失。

第三种：通过改变view的LayoutParams使得view重新布局来实现滑动




## 第四章： view 的工作原理

ViewRoot对应于ViewRootImpl类，它是连接WindowManager和DecorView的纽带，
view的三大流程均是通过ViewRoot来完成的

view的绘制流程：
	
* 1: 
	measure：测量view的宽高  getMeasuredWidth和getMeasuredHeight
	
	原始的view通过measure方法完成测量
	
	如果是viewGroup除了自己测量外，还会遍历去调用所有子元素的measure方法

* 2：layout：决定了view的四个顶点的坐标和实际view的宽高,getTop  getBottom, getLeft, getRight 拿到顶点位置

* 3：draw: 将view绘制到屏幕上
	
	绘制背景background.draw()
	
	绘制自己onDraw
	
	绘制children(dispatchDraw)
	
	绘制装饰(onDrawScrollBars)



DecorView: 就是一个FrameLayout



＝＝＝＝＝＝＝＝＝＝＝自定义View 

1：继承View重写onDraw方法  主要实现一些不规则的效果

2：继承ViewGroup派生特殊的Layout,这种方法主要用于实现自定义布局





## 第七章：Android动画深入分析
分类：

	1：view动画（平移，缩放，旋转，透明度）只支持这四种
	2：帧动画: 尽量避免使用大图片，避免OOM
	3：属性动画
属性动画的监听：

	AnimatorListener(接口) ：开始，结束，取消，重复
	AnimatorUpdateListener比较特殊，它会监听整个动画过程，每播放一帧，onAnimationUpdate就会被调用一次
	属性动画要求动画作用的对象提供该属性的set方法

### 7.3.4：对任意属性做动画：

<font size=4>
动画注意事项：
	
* 1：OOM问题  帧动画易出现
* 2：内存泄漏： 无限循环的动画，要在Activity退出时及时停止
* 3：兼容性问题：3.0一下系统
* 4：view动画的问题：view动画是对view的影像做动画，并不是真正的改变view的状态，比如setVisibility(View.GONE)失效了，这时候调用view.clearAnimation()清除view动画即可
* 5：不要使用px
* 6: 建议开启硬件加速



view动画的特殊使用场景：
LayoutAnimation : list view 中item的动画



## 第九章：四大组件的工作过程

Activity：展示型组件


Service: 计算型组件
两种状态：启动状态， 绑定状态 

BroadCastReceiver: 静态注册和动态注册， 用来实现低耦合的观察者模式，不适合用来执行耗时操作

ContentProvider: 向其他组件共享数据，需要注意的是内部的insert,update,delete,query方法需要处理好线程同步，因为这几个方法是在Binder线程池中被调用的

### 9.1 Activity的工作过程
Activity的真正实现由ActivityManagerNative.getDefault()的startActivity来完成。
ActivityManagerNative.getDefault()的具体实现是AMS（ActivityManagerService）也是一个Binder

</font>