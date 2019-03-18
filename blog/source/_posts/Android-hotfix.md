---
title: Android热修复原理
date: 2019-02-28 11:04:20
categories: "热修复"
tags:
	- "热修复"
	- "Android"
---

<font size=4 >

# 热修复原理

热修复框架技术主要有三类，代码修复，资源修复，动态链接库修复。

## 资源修复

很多资源修复的框架参考了Instant Run资源修复的原理，所以先了解一下Instant Run

### Instant Run

Instant Run是Android Studio 2.0以后新增的一个运行机制，能减少开发人员第二次及以后的构建和部署时间。

![传统编译](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2019/02/28/hotfix-01.png)

![Instant Run编译](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2019/02/28/hotfix-02.png)

* Hot Swap: 这是效率最高的部署方式，修改一个现有方法中的代码会采用这种方法。它不需要重启APP， 不需要重启Activity

* Warm Swap: App不需要重启，但是Activity需要重启。 修改或删除一个现有的资源文件时采用

* Cold Swap: App需要重启，但不需要重新安装。 添加，删除或修改一个字段和方法，添加一个类时采用该方式。

### Instant Run 资源修复的原理

* 创建新的AssetManager,通过反射调用addAssetPath方法加载外部的资源，这样新创建的AssetManager就含有了外部资源

* 将AssetManager类型的mAssets字段的引用全部替换为新的AssetManager.



## 代码修复

### 类加载方案
类加载方案基于dex分包。 为什么要dex分包，因为65535的限制和LinearAlloc限制。

* 65535限制。 
	* 主要原因是DVM ByteCode的限制，dvm指令集方法调用指令invoke-kind索引为16bits,最多能引用65535个方法数

* LinearAlloc限制
	* Dvm中的LinearAlloc是一个固定的缓冲区，当方法数超出了缓存区大小就会报错。
	
* 常用的做法：我们会新建一个hotfix分支，然后在该分支上修改bug，生成patch包，然后将patch放到Element数组dexElements的第一个元素，这样就会优先加载patch中的类，后面存在bug的类就不会被加载。 因为classloader采用的双亲委托模式，同一个类加载之后就不会再次加载	

* 类加载方案需要重启app,让classloader重新加载补丁包中的类。为什么要重启？ 因为类是无法卸载的，想要重新加载新的类，就需要重启app. 所以采用类加载方案是不能及时生效的。
	* Q	Q空间超级补丁，Nuwa,就是按照上面将补丁包放到Element数组的首位优先加载
	* 微信的Tinker将新旧APK做了diff,得到patch.dex,再将patch.dex与手机APK中的classes.dex做合并，生成新的classes.dex.然后在运行时通过反射将classes.dex放到Element数组的第一位。

### 底层替换方案
底层替换方案不会再次加载新类，而是直接在Native层修改原有类。

* 底层替换方案主要是替换ArtMethod结构体中的字段或者替换整个ArtMethod结构体。
* AndFix采用的是替换ArtMethod结构体中的字段，但是会有兼容性问题，因为厂商会修改结构体，导致替换失败
* Sophix采用的是替换整个ArtMethod结构体，不会存在兼容问题。
* 底层替换方案是可以及时生效的，不需要重启。


### Instant Run方案

* 主要是通过ASM来操作代码，进行方法的修改。
* ASM是一个字节码操作框架，能够动态生成类或者增强现有类的功能。ASM可以直接产生class文件，也可以在类被加载到虚拟机之前动态改变类的行为。


## 动态链接库修复

这个主要是更新so,也就是重新加载so. 加载so有两个方法

* System的load方法. 该方法传入的参数是so在磁盘空间的完整路径

* System的loadLibrary方法。 该方法传入的参数是so的名称，用于加载apk安装后自动从APK包中复制到/data/data/packagename/lib下的so. 目前so的修复主要是针对这两个方法。

so修复的两个方案

1. 将so补丁插入到NativeLibraryElement数组的前部，让so补丁优先加载
2. 调用System的load方法来接管so的加载入口。



参考：

《Android进阶解密》   
 