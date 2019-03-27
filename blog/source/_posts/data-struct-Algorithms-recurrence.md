---
title: 递归：如何用三行代码找到“最终推荐人”？
date: 2019-02-02 16:32:46
tags:
	- "数据结构与算法" 
	
categories: "数据结构与算法"	
---

<font size=4>

###  本文是极客时间 《数据结构与算法》章节学习的笔记，仅供自己记录理解

## 理解递归

递归的关键就是找到如何将大问题分解为小问题的规律，并且基于此写出递推公式，然后再推敲终止条件，最后根据递推公式和终止条件写成代码。

例如： 

		f(n)=f(n-1)+1 其中，f(1)=1
		
		
		
转换为：
		
		int f(int n) { 
			if (n == 1) return 1;
			return f(n-1) + 1;
		}


## 递归需要满足的三个条件

* 一个问题的解可以分解为几个子问题的解
* 这个问题与分解之后的子问题，除了数据规模不同，求解思路完全一样
* 存在递归终止条件

编写递归代码的关键是，只要遇到递归，我们就把它抽象成一个递推公式，不用想一层层的调用关系，不要试图用人脑去分解递归的每个步骤。


## 优缺点

* 优点： 代码简洁高效

* 缺点：栈溢出、重复计算、函数调用耗时多、空间复杂度高等


## 怎么将递归代码改写为非递归代码？

* 采用 迭代循环 这种方法。 

例如 f(x) =f(x-1)+1 这个递推公式。改写：

		int f(int n) {
			int ret = 1;
				for (int i = 2; i <= n; ++i) {
					ret = ret + 1;
				}
			return ret;
		}



## 如何找到“最终推荐人”？解决方案是这样的：

	long findRootReferrerId(long actorId) {
		Long referrerId = select referrer_id from [table] where actor_id = actorId;
		if (referrerId == null) return actorId;
		return findRootReferrerId(referrerId);
	}

但是上述代码并不能工作，因为

* 如果递归很深，可能会有堆栈溢出的问题（解决的方法：可以用限制递归深度来解决）
* 如果数据库里存在脏数据，我们还需要处理由此产生的无限递归问题。比如 demo 环境下数据库中，测试工程师为了方便测试，会人为地插入一些数据，就会出现脏数据。如果 A 的推荐人是 B，B 的推荐人是 C，C 的推荐人是 A，这样就会无限循环。（解决方法：就是自动检测 A-B-C-A 这种“环”的存在）


## 思考

* 我们平时调试代码喜欢使用 IDE 的单步跟踪功能，像规模比较大、递归层次很深的递归代码，几乎无法使用这种调试方式。对于递归代码，有什么好的调试方法呢？

	* 调试递归: 1.打印日志发现，递归值。2.结合条件断点进行调试。

