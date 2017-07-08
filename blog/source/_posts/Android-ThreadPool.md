---
title: Android的线程和线程池
date: 2017-07-09 00:31:10
tags: "学习笔记"
categories: "Android"
---
## Android的线程和线程池

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
    	

参考：
《Android开发艺术探索》