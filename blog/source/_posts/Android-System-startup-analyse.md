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

### init进程启动总结
* 创建和挂载启动所需的文件目录
* 初始化和启动属性服务
* 解析init.rc配置文件，并启动Zygote进程


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
	

* zygote总结：
	* 创建AppRuntime,并调用start()方法，启动zygote进程
	* 创建java虚拟机，并为java虚拟机注册jni方法
	* 通过jni调用zygoteinit中main方法，进入zygote的java框架层(之前都是在native本底层)
	* 开辟一个进程处理启动systemserver，来处理系统进程（创建系统进程）
	* 通过registerZygoteSocket方法创建服务器端socket, 然后通过无限循环监听AMS的请求，收到之后创建新的应用进程

	


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

### 4.Launcher启动过程

Launcher就是Android系统的桌面，其特点如下：

 * 作为android系统的启动器，用于启动应用程序。
 * 用于显示和管理应用程序快捷图标，或者其他桌面组件。

被Systemserver进程启动的AMS会启动Launcher,Launcher启动后会将应用程序快捷图标显示到桌面上.



![Android系统启动流程](http://upload-images.jianshu.io/upload_images/5944729-2c1f6add3384bd4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

