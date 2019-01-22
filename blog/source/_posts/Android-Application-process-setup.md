---
title: Android应用程序进程启动过程
date: 2019-01-21 16:18:45
tags:
	- "Android系统" 
	- "操作系统"
	

categories: "Android"
---

<font size=3>


## 应用程序进程启动介绍
### AMS发送启动应用程序进程请求

![AMS请求Zygote](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2019/01/21/ams-zygote.png)



### Zygote接受请求并创建应用程序进程

![](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2019/01/21/zygote-activitythread.png)


* AMS与Zygote之间是通过socket通讯的。 
* 因为Zygote通过registerZygoteSocket方法创建服务器端socket, 然后通过无限循环监听AMS的请求，收到之后创建新的应用进程	


### binder线程池启动
### 消息循环创建
ActivityThread

* 创建主线程Looper. Looper.prepareMainLooper()
* 创建主线程H类(继承Handler)
* Looper循环。 Looper.loop();