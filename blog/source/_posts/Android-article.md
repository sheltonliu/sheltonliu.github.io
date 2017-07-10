---
title: Android开发艺术探索学习笔记
date: 2017-07-05 23:45:32
tags:
	- "Android开发艺术探索笔记"
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


#### IPC方式的优缺点和适用场景
AIDL: Android Interface definition language   

|   | 优点  | 缺点  | 试用场景 |
|:--------------|:------------- |:---------------:| -------------:|
| Bundle        | 简单易用       |  只传输bundle支持的类型  | 四大组建通讯 |
| 文件共享        | 简单易用      |  不适合高并发,无法进程间即时通讯  | 无并发访问实时性不高的场景|
| AIDL |支持一对多并发,支持实时通讯 |使用稍复杂,需要处理好线程同步 | 一对多通讯且有RPC需求|
| Message |   支持一对多串行 , 支持实时通讯 | 不能很好处理高并发情形，不支持RPC | 低并发无RPC需求|
| ContentProvider |数据源访问强大，支持一对多并发共享  | 受约束的AIDL主要提供数据源的CRUD操作 |一对多的进程间的数据共享  |
| socket |通过字节流传输，支持一对多并发实时通讯  |细节繁琐，不支持直接的RPC | 网络数据交换 |




## 第三章：View的事件体系
### 1.View基础知识
#### 1.1 View与ViewGroup的关系

* 1.View是Android中控件的基类
* 2.ViewGroup名字上翻译是控件组，ViewGroup内部包含许多控件，ViewGroup继承View意味着View可以是单个控件也可以是由多个控件组成的一组控件.如LinearLayout不单是view还是ViewGroup.

#### 1.2.MotionEvent和TouchSlop
* MotionEvent: ACTION_DOWN, ACTION_MOVE, ACTION_UP.通过MotionEvent可以得到点击事件的X,Y坐标。有两组方法
	* getX/getY:返回相对于当前View左上角的X,Y坐标
	* getRawX/getRawY:返回相对于手机屏幕左上角的X,Y坐标。

* TouchSlop：系统所识别的滑动的最小距离，是一个常量，和设备有关。不同设备上值是不同的。

#### 1.3 VelocityTracker,GestureDetector,Scroller
* VelocityTracker：用于追踪手指滑动的速度。获取速度之前先computeCurrentVelocity计算速度。不需要的时候需要
				
			velocityTracker.clear();
			velocityTracker.recycle();
			
* GestureDetector:手势检测			
* Scroller：弹性滑动对象。实现View的弹性滑动

### 2.View的滑动
#### 2.1使用scrollTo/scrollBy
#### 2.2使用动画
View平移，或采用属性动画
#### 2.3改变布局参数
如marginLeft

### View的事件分发机制
当一个点击事件产生后，流程如下：
Activity->Window->顶级View->处理分发事件。
事件首先由Activity的dispatchTouchEvent进行派发，具体工作由Window来完成，Window会将事件传递给DecorView,DecorView是当前页面的底层容器(即setContentView所设置的View的父容器)，然后由该顶级View处理分发。

如一个Activity的布局如下：
![Activity布局](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/08/Activity%E5%B8%83%E5%B1%80.jpg)
事件流程如下：

![事件分发流程](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/08/View%E4%BA%8B%E4%BB%B6%E6%B5%81%E7%A8%8B.jpg)

### 重点：其事件传递的核心规则，可由下面的伪代码来表示：
	
	public boolena dispatchTouchEvent(MotionEvent ev){
		boolena consume = false;
		if (onInterceptTouchEvent(ev)){
			consume = onTouchEvent(ev);
		}else{
			consume = child.dispatchTouchEvent(ev);
		}
		return consume; 
	}
了解了伪代码的流程，也就了解了事件传递的流程


### 3.滑动冲突的解决方式： 
* 外部拦截法：重写父容器的onInterceptTouchEvent 方法

* 内部拦截法：需要重写子元素的dispatchTouchEvent方法并配合requestDisallowInterceptTouchEvent方法来用	
	


## 第四章： view 的工作原理

ViewRoot对应于ViewRootImpl类，它是连接WindowManager和DecorView的纽带，
view的三大流程均是通过ViewRoot来完成的

view的绘制流程：
	
* 1: measure：测量view的宽高  getMeasuredWidth和getMeasuredHeight
	
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

<font size=3>
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



## 第十章 Android消息机制

### 10.1:Android消息机制概述

* Android的消息机制主要是指Handler的运行机制。Handler需要底层的MessageQueue和Looper支撑。

* MessageQueue: 单链表数据结构，是消息存储单元，不能处理消息

* Looper: 无线循环查找是否有新消息，有的话就处理，没有就一直等待
注意Looper是运行在创建Handler所在的线程中的。

* Handler通过ThreadLocal获取每个线程的Looper来构建消息循环系统。

* 线程默认是没有Looper的。如果要使用Handler，就需要为线程创建Looper

* Android主线程也就是UI线程：ActivityThread在创建的时候就会初始化Looper.这也就是Android主线程默认可以使用Handler的原因。

* 系统为什么不允许在子线程中访问UI？ 因为Android的UI控件不是线程安全的。

* 为什么UI不采用加锁机制？
	* 1：加锁会让UI访问逻辑复杂
	* 2：会让UI访问效率降低

### 10.2:Android消息机制分析

#### 1：MessageQueue工作原理
* MessageQueue单链表结构，在插入和删除中有优势
* enqueueMessage:插入
* next : 读取并移除数据。next方法是无限循环的方法，队列中没有消息该方法会一直阻塞，有消息就返回并将该数据移除

#### 2:Looper工作原理

Handler所在线程需要Looper,没有Looper线程会报错。如何在线程中创建Looper呢？ 

	new Thread("threadname"){
		@Override
		public void run(){
			Looper.prepare(); //为当前线程创建Looper
			Handler handler = new Handler();
			Looper.loop();   //开启消息循环
		}
	}
* prepareMainLooper:给主线程创建Looper使用的
* getMainLooper:  获取主线程Looper
* quit: 直接退出Looper
* quitSafely: 把队列中已有消息处理完毕后安全退出
* 如果在子线程中创建Looper,在事情处理完后调用quit终止消息循环，否则子线程一直处于等待状态。退出Looper后线程就会立刻终止。
	
#### 3:Handler工作原理
主线程消息循环模型：
ActivityThread通过ApplicationThread和AMS进行进程间通讯。AMS以进程间通讯的方式完成ActivityThread的请求后回调ApplicationThread中的Binder方法，然后ApplicationThread向H发送消息，H将逻辑切换到ActivityThread中执行，即切换到主线程中执行。

![Android消息机制](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/06/Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6.jpeg)

![Android消息机制2](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/06/Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B62.jpg)




## 第十一章 Android的线程和线程池

### Android中的线程形态
#### 1.AsyncTask

AsyncTask封装了Thread和Handler,是抽象的泛型类，提供了Params,Progress,Result这三个泛型参数


#### 四个核心方法：

* 1.onPreExecute() 主线程中执行，异步任务之前调用
* 2.doInBackground(Params... par) 异步后台执行
* 3.onProgressUpdate(Progress... value) 主线程中执行，进度发生变化该方法被调用
* 4.onPostExecute(Result res) 主线程中执行，异步任务执行完改方法被调用

#### 使用过程中有以下条件限制

* 1.AsyncTask的类必须在主线程中加载。在5.0的源码中ActivityThread的main方法中调用AsyncTask的init方法
* 2.AsyncTask的对象必须在主线程中创建
* 3.execute方法必须在UI线程调用
* 4.不要在程序中直接调用上述四个核心方法
* 5.一个AsyncTask对象只能执行一次，即只能调用一次execute方法，否则报运行时异常
* 6.版本差异
	* 1.在Android1.6之前是串行执行任务；
	* 2.Android1.6的时候采取并行处理；
	* 3.Android3.0又采取串行执行，尽管如此3.0以后的版本还可以通过AsyncTask的executeOnExecutor来并行执行任务

#### 2.HandlerThread
HandlerThread继承了Thread. 该线程运行的时候创建Looper。可以说是自带Looper的Thread

	
#### 3.IntentService
IntentService抽象类，继承了Service. 内部包含了ServiceHandler和HandlerThread.

![IntentService流程1](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/06/IntentService%E6%B5%81%E7%A8%8B1.jpg)

![IntentService流程2](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/06/IntentService%E6%B5%81%E7%A8%8B2.jpg)

	public class LocalIntentService extends IntentService {
    private static final String TAG = "LocalIntentService";

    public LocalIntentService() {
        super(TAG);
    }

    @Override
    protected void onHandleIntent(final Intent intent) {
        final String action = intent.getStringExtra("task_action");
        Log.d(TAG, "receive task :" +  action);

        SystemClock.sleep(3000);
        if ("com.ryg.action.TASK1".equals(action)) {
            Log.d(TAG, "handle task: " + action);
        }
    }

    @Override
    public void onDestroy() {
        Log.d(TAG, "service destroyed.");
        super.onDestroy();
    }
	}
	
启动服务
	 
	 Intent service = new Intent(this, LocalIntentService.class);
    service.putExtra("task_action", "com.ryg.action.TASK1");
    startService(service);
    service.putExtra("task_action", "com.ryg.action.TASK2");
    startService(service);
    service.putExtra("task_action", "com.ryg.action.TASK3");
    startService(service);
    
### Android中的线程池
线程池的优点：
	
* 1.重用线程，避免开销
* 2.有效控制线程并发数，避免因抢占资源造成的阻塞
* 3.有效管理，提供定时及循环间隔执行等功能

#### ThreadPoolExecutor
参数：

* 1.corePoolSize: 核心线程数。默认情况下核心线程在线程池中会一直存活，即便是闲置状态。如果将ThreadPoolExecutor的allowCoreThreadTimeOut设置为true,那么闲置的核心线程在等待新任务到来时会有超时策略
* 2.maximumPoolSize：容纳的最大线程数
* 3.keepAliveTime：非核心线程闲置时的超时时长，超过这个时长，非核心线程就会被回收。
* 4.TimeUnit
* 5.workQueue
* 6.ThreadFactory

#### 线程池的分类
* 1.FixedThreadPool: newFixedThreadPool方法创建。是一种数量固定的线程池，只有核心线程，且核心线程不会被回收，可以快速相应外界请求。没有超时机制，没有队列大小限制。
* 2.CachedThreadPool：newCachedThreadPool方法创建。是一种线程数量不定的线程池，只有非核心线程。有超时机制。适合执行大量的耗时较少的任务     
* 3.ScheduledThreadPool：核心线程数量是固定的，非核心线程数是没有固定的，当非核心线程闲置时会被立即回收。主要用于执行定时任务和具有固定周期的重复任务。
* 4.SingleThreadExecutor：线程池内部只有一个核心线程，确保所有的任务都在同一个线程中按顺序执行。

	
		private void runThreadPool() {
        	Runnable command = new Runnable() {
            @Override
            public void run() {
                SystemClock.sleep(2000);
            }
        	};

        	ExecutorService fixedThreadPool = Executors.newFixedThreadPool(4);
        	fixedThreadPool.execute(command);
        
        	ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        	cachedThreadPool.execute(command);
        
        	ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(4);
        	// 2000ms后执行command
        	scheduledThreadPool.schedule(command, 2000, TimeUnit.MILLISECONDS);
        	// 延迟10ms后，每隔1000ms执行一次command
        	scheduledThreadPool.scheduleAtFixedRate(command, 10, 1000, TimeUnit.MILLISECONDS);

        	ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
        	singleThreadExecutor.execute(command);
    	}
    	
    	
    	
    	
## 第十五章 Android性能优化

性能优化这是个很大的范畴。

1：布局优化： <include> <merge>(减少层级) ViewStub 

2: 绘制优化：指View的onDraw方法避免大量操作
3. Bitmap优化，线程优化

一些性能优化建议：

* 1.避免创建过多对象
* 2.不要过多使用枚举，枚举占用的内存空间比整型大
* 3.常量使用static final来修饰
* 4.使用Android特有数据结构比如SparseArray和Pair等。
* 5.适当使用软引用(SoftReference)和弱引用(WeakReference)
* 6.尽量采用静态内部类，避免潜在的由于内部类导致的内存泄漏。


    



















