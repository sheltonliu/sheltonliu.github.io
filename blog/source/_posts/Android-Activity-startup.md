---
title: Activity启动过程分析
date: 2019-01-23 09:52:30
tags:
	- "Android系统" 
	- "操作系统"
	
categories: "Android"	
---

<font size=3>

## Activity启动过程分析

这里的Activity分为根Activity和普通Activity.这两种启动

* 所谓根Activity启动就是类似冷启动，先要启动该Activity所在的应用程序进程，然后在该进程内启动Activity
* 普通Activity的启动就是，前提它们所在的进程已经启动了，然后就是Activity之间的跳转之类的。

### 根Activity启动的整体流程如下:
![根Actiivty启动整体流程](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2019/01/23/01.png)

Launcher到AMS的调用过程如下：

![Launcher-AMS](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2019/01/23/Launcher-AMS.png)

因为AMS到zygote进程，以及zygote进程启动应用程序，这些在上篇文章有介绍，这里不再重复了 [应用程序进程启动介绍](https://www.jianshu.com/p/364536beb16d)

接下来就是AMS与ApplicationThread之间的调用

![AMS-ApplicationThread](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2019/01/23/AMS-ApplicationThread.png)

AMS与ApplicatinThread之间是通过binder通讯的. 



最后就是ApplicationThread与ActivityThread之间的调用

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2019/01/23/AppThread-Activity.png)


### 普通Activity的启动

就只有两层调用

* AMS与ApplicationThread之间的调用
* ApplicationThread与ActivityThread之间的调用

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2019/01/23/systemserver-process.png)



		private class ApplicationThread extends IApplicationThread.Stub {
			....
			....
		}

ApplicationThread是ActivityThread中的内部类，运行在binder线程池中
ApplicationThread是AMS与应用程序进程之间的通许桥梁

参考：Android进阶解密
		





