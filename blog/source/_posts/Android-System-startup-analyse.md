---
title: Android系统启动过程分析
date: 2019-01-14 14:14:55
tags:
	- "Android系统" 
	- "操作系统"
categories: "Android"
---

<font size=3>

## 第一个系统进程init
* Android设备的启动必须经历三个阶段：Boot Loader, Linux Kernel和 Android系统服务。 默认情况下他们都有自己的启动画面。严格说Android系统是运行在Linux内核之上的一系列"服务进程"，并不是完整意义的"操作系统"。这些进程是维持设备正常工作的关键，它们的"老祖宗"就是init.

* 作为Android中第一个被启动的进程，init的pid=0.它通过解析init.rc脚本来构建出系统形态。

### init.rc语法
一个完整的init.rc脚本由4种类型组成：

* Action(动作)
* Commands（命令）
* Services(服务)
* Options(选项)

## 系统关键服务的启动简析
* init作为Android系统的第一个进程，它通过解析init.rc来陆续启动其他关键系统进程。 这其中最重要的就是ServiceManager, Zygote, SystemServer.

### 1.Android的"DNS服务器" ServiceManager

* ServiceManager是在init.rc里描述并由init进程启动。 

		/*system/core/rootdir/Init.rc*/
		service servicemanager /system/bin/servicemanager
		class core
		user system 
		group system
		critical (该选项说明是系统关键进程)
		onrestart restart zygote
		onrestart restart media
		onrestart restart surfaceflinger
		onrestart restart drm

* servicemanager是一个linux程序。它在设备中的存储路径是/system/bin/servicemanager. 源码路径是/frameworks/native/cmds/servicemanager.

* ServiceManager所属class是core. core组的特性是，这些进程会同时被启动或停止。critical (该选项说明是系统关键进程)意味着如果进程在4分钟内异常退出超过4次，则设备将重启进入还原模式。当ServiceManager重启时，其他关键进程如zygote,media,surfaceflinger等也会被restart.



### 2."孕育"新的线程和进程 Zygote

* zygote 字面意思是"受精卵"，可以”孕育“一个”新生命“.Android中的大多数应用进程和系统进程都是通过zygote来生成的。




zygote也是由init在解析rc脚本时启动的。
		
		ServiceName:zygote
		Path: /system/bin/app_process
		Arguments: -Xzygote /system/bin --zygote --start-system-server
		
所在的程序名叫"app_process".源码路径在/frameworks/base/cmds/app_process  看看它的Android.mk
		
		LOCAL_SRC_FILES:= \
			  app_main.cpp
		LOCAL_SHARED_LIBRARIES := \
				libcutils \
				libutils \
				liblog \
				libbinder \
				libandroid_runtime
		LOCAL_MODULE:= app_process
		
得知app_process具体实现是app_main.cpp. 其内容主要是：

		if (zygote){
		    //启动虚拟机，并执行ZygoteInit
			runtime.start("com.android.internal.os.ZygoteInit",startSystemServer? "start-system-server" : "");
		}
		

runtime是一个变量，它实际上是一个AndroidRuntime对象。其start函数源码如下
	
	/*frameworks/base/core/jni/AndroidRuntime.cpp*/
	void AndroidRuntime::start(const char* className, const char* options)
	......
	JNIEnv* env;
	if (startVm(&mJavaVm, &env) != 0 ){//启动虚拟机
		return;
	}
	onVmCreated(env); //虚拟机启动后的回调
	

* zygote的作用：
	* 其具体执行是通过zygoteInit来处理的。ZygoteInit是运行在java虚拟机之上的。 
	* 开辟一个进程处理启动systemserver，来处理系统进程（创建系统进程）
	* 完成上面操作之后，会进入无线循环来处理客户端相应。如果接受到新的应用，则开辟新的应用进程来处理。 （创建应用进程）



### 3.Android的"系统服务"-- SystemServer

SystemServer是Android进入Launcher前的最后准备，它提供了众多由java语言编写的"系统服务"。

* ZygoteInit通过Zygote.forkSystemServer来生成一个新进程，用于承载各个系统服务
* native本地层Service(比如SurfaceFlinger, AudioFlinger等)的启动。
* java层，各个service的启动
	* 创建新的线程来启动
			
			class ServerThread extends Thread {
				public void run(){
					Looper.prepareMainLooper();
					//启动各个系统服务，如：PowerManagerService、ActivityManagerService等
					Looper.loop();
				}
			} 		

后续会分析具体的系统服务。ActivityManagerService是导致Launcher被启动的关键，后面会在分析的。



	

		
		


					  
			
		
		


 

