---
title: Android View的事件体系
date: 2017-07-09 00:21:41
tags: "学习笔记"
categories: "Android"
---

## View的事件体系
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

参考：
《Android开发艺术探索》