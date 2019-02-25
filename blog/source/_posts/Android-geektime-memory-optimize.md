---
title: Android内存优化
date: 2019-02-13 11:56:46
tags:
	- "Android" 
	- "Android优化相关"
	
categories: "Android"	
---

<font size=3>

## Android内存优化

### 移动设备的发展

Facebook 有一个叫device-year-class的开源库，它会用年份来区分设备的性能

手机运行内存（RAM）其实相当于我们的 PC 中的内存
手机不使用 PC 的 DDR 内存，采用的是 LPDDR RAM，全称是“低功耗双倍数据速率内存”，其中 LP 就是“Lower Power”低功耗的意思。


### 由内存引起的两个问题

* 异常
	* java堆 : oom
	* PSS(物理内存)： 重启
	* VSS(虚拟内存)： oom,内存分配失败(tgkill)
	
* 卡顿
	*　GC
	* low memory killer
	


### 内存的误区

* 回顾一下 Android Bitmap 内存分配的变化
	* 在 Android 3.0 之前，Bitmap 对象放在 Java 堆，而像素数据是放在 Native 内存中。如果不手动调用 recycle，Bitmap Native 内存的回收完全依赖 finalize 函数回调，但是这个时机不太可控。
	
	* Android 3.0～Android 7.0 将 Bitmap 对象和像素数据统一放到 Java 堆中，这样就算我们不调用 recycle，Bitmap 内存也会随着对象一起被回收。不过 Bitmap 是内存消耗的大户，把它的内存放到 Java 堆中存在的问题是
		* 大量占用java堆内存，导致OOM
		* 放到java堆可能会引起大量的GC
		* 无法充分利用物理内存
		
	* 有没有一种实现，可以将 Bitmap 内存放到 Native 中，也可以做到和对象一起快速释放，同时 GC 的时候也能考虑这些内存防止被滥用？NativeAllocationRegistry 可以一次满足你这三个要求，Android 8.0 正是使用这个辅助回收 Native 内存的机制，来实现像素数据放到 Native 内存中。Android 8.0 还新增了硬件位图 Hardware Bitmap，它可以减少图片内存并提升绘制效率。
	


* 误区一：内存占用越少越好
	* 应用是否占用了过多的内存，跟设备、系统和当时情况有关，而不是 300MB、400MB 这样一个绝对的数值
	* 希望可以做到“用时分配，及时释放”
	
* 误区二：Native 内存不用管
	* 当系统物理内存不足时，lmk 开始杀进程，从后台、桌面、服务、前台，直到手机重启。
	
	* 我们比较熟悉的是 Fresco 图片库在 Dalvik 会把图片放到 Native 内存中。事实上在 Android 5.0～Android 7.0，也能做到相同的效果，只是流程相对复杂一些。
					
		
			// 步骤一：申请一张空的 Native Bitmap
			Bitmap nativeBitmap = nativeCreateBitmap(dstWidth, dstHeight, nativeConfig, 22);

			// 步骤二：申请一张普通的 Java Bitmap
			Bitmap srcBitmap = BitmapFactory.decodeResource(res, id);

			// 步骤三：使用 Java Bitmap 将内容绘制到 Native Bitmap 中
			mNativeCanvas.setBitmap(nativeBitmap);
			mNativeCanvas.drawBitmap(srcBitmap, mSrcRect, mDstRect, mPaint);

			// 步骤四：释放 Java Bitmap 内存
			srcBitmap.recycle();
			srcBitmap = null；

		存在的问题就是一个是兼容性问题，另外一个是频繁申请释放 Java Bitmap 容易导致内存抖动。
	
	
	
## 测量方法

### java内存分配

* 这个时候最常用的有 Allocation Tracker 和 MAT 这两个工具。


### Native内存分配

* [Malloc调试](https://android.googlesource.com/platform/bionic/+/master/libc/malloc_debug/README.md)可以帮助我们去调试 Native 内存的一些使用问题，例如堆破坏、内存泄漏、非法地址等。Android 8.0 之后支持在非 root 的设备做 Native 内存调试，不过跟 AddressSanitize 一样，需要通过[wrap.sh](http://developer.android.com/ndk/guides/wrap-script.html)做包装。

		adb shell setprop wrap.<APP> '"LIBC_DEBUG_MALLOC_OPTIONS=backtrace logwrapper"'

* [Malloc钩子](http://android.googlesource.com/platform/bionic/+/master/libc/malloc_hooks/README.md)是在 Android P 之后，Android 的 libc 支持拦截在程序执行期间发生的所有分配 / 释放调用，这样我们就可以构建出自定义的内存检测工具。

		adb shell setprop wrap.<APP> '"LIBC_HOOKS_ENABLE=1"'



## 内存优化探讨

### 1.设备分级

* 内存优化首先需要根据设备环境来综合考虑

		if (year >= 2013) {
    		// Do advanced animation
		} else if (year >= 2010) {
    		// Do simple animation
		} else {
    		// Phone too slow, don't do any animations
		}


* 缓存管理。我们需要有一套统一的缓存管理机制。 当“系统有难”时，也要义不容辞地归还。我们可以使用 OnTrimMemory 回调，根据不同的状态决定释放多少内存。统一缓存管理可以更好地监控每个模块的缓存大小。

* 进程模型。一个空的进程也会占用 10MB 的内存，而有些应用启动就有十几个进程，甚至有些应用已经从双进程保活升级到四进程保活，所以减少应用启动的进程数、减少常驻进程、有节操的保活，对低端机内存优化非常重要。

* 安装包大小。安装包中的代码、资源、图片以及 so 库的体积，跟它们占用的内存有很大的关系。一个 80MB 的应用很难在 512MB 内存的手机上流畅运行。这种情况我们需要考虑针对低端机用户推出 4MB 的轻量版本，例如 Facebook Lite、今日头条极速版都是这个思路。

### 2.Bitmap优化

* 方法一，统一图片库。
	
	* 图片内存优化的前提是收拢图片的调用，这样我们可以做整体的控制策略。例如低端机使用 565 格式、更加严格的缩放算法，可以使用 Glide、Fresco 或者采取自研都可以。而且需要进一步将所有 Bitmap.createBitmap、BitmapFactory 相关的接口也一并收拢。
	

* 方法二，统一监控。
	* 大图片监控
	* 重复图片监控
	* 图片总内存
		* 在 OOM 崩溃的时候，也可以把图片占用的总内存、Top N 图片的内存都写到崩溃日志中，帮助我们排查问题。
		


### 3. 内存泄漏

* Java 内存泄漏
	* 类似 LeakCanary 自动化检测方案， 或者腾讯 ResourceCanary方案:[腾讯APM监控](https://github.com/Tencent/matrix)
	* Activity 泄漏，即因为各种原因导致 Activity 被生命周期远比该 Activity 长的对象直接或间接以强引用持有，导致在此期间 Activity 无法被 GC 机制回收的问题.  
		* Activity: 匿名内部类隐式持有外部类的引用导致的泄漏
		
	* Bitmap 分配及回收追踪
		* 发现问题最多最突出的，是缓存的滥用问题，最为典型的是使用 static LRUCache 来缓存大尺寸 Bitmap
		
				private static LruCache<String, Bitmap> sBitmapCache = new LruCache<>(20);
				public static Bitmap getBitmap(String path) {
    				Bitmap bitmap = sBitmapCache.get(path);
    					if (bitmap != null) {
        					return bitmap;
    					}
					bitmap = decodeBitmapFromFile(path);
    				sBitmapCache.put(path, bitmap);
    				return bitmap;
				}	
				
		* 比如上面的代码，作用是缓存一些重复使用的 Bitmap 避免重复解码损失性能，但由于 sBitmapCache 是静态的且没有清理逻辑，缓存在其中的图片将永远无法释放，除非 20 个的配额用尽或图片被替换。LruCache 对缓存对象的 个数 进行了限制，但没有对对象的 总大小 进行限制. 因此如果缓存里面存放了数个大图或者长图，将长期占用大量内存
				
	* 线程监控
		* 常见的 OOM 情况大多数是因为内存泄漏或申请大量内存造成的，比较少见的有下面这种跟线程相关情况，但在我们 crash 系统上有时能发现一些这样的问题。
				
				java.lang.OutOfMemoryError: pthread_create (1040KB stack) failed: Out of memory
			
			OutOfMemoryError 这种异常根本原因在于申请不到足够的内存造成的，直接的原因是在创建线程时初始 stack size 的时候，分配不到内存导致的。
			
		* 可以看到每个线程初始化都需要 mmap 一定的 stack size，在默认的情况下一般初始化一个线程需要 mmap 1M 左右的内存空间
		* 对线程数量的限制，可以一定程度避免 OOM 的发生
	
	

### 内存监控

	long javaMax = runtime.maxMemory();
	long javaTotal = runtime.totalMemory();
	long javaUsed = javaTotal - runtime.freeMemory();
	// Java 内存使用超过最大限制的 85%
	float proportion = (float) javaUsed / javaMax;



* GC监控 

		// 运行的 GC 次数
		Debug.getRuntimeStat("art.gc.gc-count");
		// GC 使用的总耗时，单位是毫秒
		Debug.getRuntimeStat("art.gc.gc-time");
		// 阻塞式 GC 的次数
		Debug.getRuntimeStat("art.gc.blocking-gc-count");
		// 阻塞式 GC 的总耗时
		Debug.getRuntimeStat("art.gc.blocking-gc-time");






参考： [微信 Android 终端内存优化实践](https://mp.weixin.qq.com/s/KtGfi5th-4YHOZsEmTOsjg?)		
				

https://time.geekbang.org/column/article/71277

https://time.geekbang.org/column/article/71610

				
	
	
	
			
	
	
		
		
















