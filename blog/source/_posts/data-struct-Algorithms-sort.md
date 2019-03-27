---
title: 排序
date: 2019-03-04 15:49:44
tags:
	- "数据结构与算法" 
	
categories: "数据结构与算法"	
---

<font size=4>


     
              | 是原地排序?  | 是否稳定？ | 最好,最坏,平均
------------- | ----------  | ---------| ------------
冒泡排序       | Y   | Y | O($n$),   O($n^2$) ,  O($n^2$) 
插入排序       | Y   | Y | O($n$),   O($n^2$) ,  O($n^2$) 
选择排序       | Y   | N | O($n^2$) ,O($n^2$) ,  O($n^2$) 
归并排序       | N 空间复杂度O(n)   | Y | O($nlogn$) ,O($nlogn$),O($nlogn$) 
快速排序       | Y   | N | O($nlogn$) ,O($n^2$) ,  O($nlogn$) 


## 冒泡排序

冒泡排序只会操作相邻的两个数据，每次冒泡操作都会对相邻的两个元素进行比较，看是否满足大小关系要求，如果不满足就互换。 一次冒泡会至少让一个元素移动到它应该在的位置，重复n次，就完成了n个数据排序工作


## 插入排序

* 首先我们将数组中的数据分为两个区间， **已排序区间** 和**未排序区间** 
* 初始已排序区间只有一个元素，就是数组的第一个元素。 
* 核心思想是取未排序区间中的元素，在已排序区间中找到合适的位置进行插入， 并保证已排序区间数据一直有序。
* 重复这个过程，直到未排序空间为空


## 选择排序

* 算法的思路类似插入排序，也分为**已排序区间** 和**未排序区间** 
* 每次从未排序空间中选择最小的元素，将其放到已排序区间的末尾。


## 问题
 
 
冒泡排序和插入排序时间复杂度都是O($n^2$), 且都是原地排序算法，为什么插入排序要比冒泡排序更受欢迎？

* 冒泡的数据交换要比插入排序的数据移动要复杂



## 归并排序
* 用到了分而治之的思想，递归的编程技巧。 
* 如果要排序一个数组， 我们先把数组从中间分成前后两部分，然后每一部分在从中间分成两部分，这样递归。分解完之后进行合并。


 
## 快速排序
* 用到了分而治之的思想
* 如果要排序数组中下标从p到r之间的一组数据，我们选择p到r之间的任意一个数据为pivot(分区点)
* 我们遍历p到r之间的数据，将小于pivot的放到左边，将大于pivot的放到右边，将pivot放到中间。  然后根据递归，在左边和右边数组中再选择pivot点进行拆分， 最后进行合并 

分区算法(优化)

* 三数取中法。 
	* 我们从区间的首尾中间，分别取出一个数，取这三个数的中间值作为分区点。如果排序的数组较大，可能要“五数取中”，或“十数取中”

* 随机法： 在排序的区间中，随机选择一个元素作为分区点	
	




