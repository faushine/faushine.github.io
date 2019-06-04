---
layout: post
title: "一个有趣的算法——QuickSelect"
date:   2015-04-01
author: Faushine
tags: 
- Algorithm
---

其实这是一道算法课的作业，方法非常巧妙，希望我能叙述清楚。
> Question:
> Find the k-th smallest element in an unsorted array of length n in O(n)

任何一种排序算法都不能达到O(n)的时间复杂度，因此不能通过排序找到第k个最大值。QuickSelect是QuickSort的衍生，利用QuickSort中的partition达到分治的目的。

关于这个算法详细的说明可以查看[wikipedia-QuickSelect](https://en.m.wikipedia.org/wiki/Quickselect)

QuickSort中的partition算法是把一串数列分成两拨，即以关键字为分界点，小于它的元素在一边，大于它的元素在一边。它的作用是相当重要的，比如你要在10000个元素中查找第4000个最小值，而若知道关键字在第5000位，那么就不必再去查找另一半的元素了。

举个例子说明这个算法的具体过程：

2,7,8,9,11,3,5,14,6,10,12,1,19,32,17 现在有十五个尚未排序的元素，希望查找到第3个最小值

(1) 分组。（每五个元素一组，不要问我为什么，统计表明当五个一组的时间复杂度最低）

2,7,8,9,11  3,5,14,6,10  12,1,19,32,17

(2) 求出每组的中位数，放入新数组median 中。

8  6  17

(3) 在median 中找到n/10个最小值取上限（即第二个）。

8

(4) 将所有元素数组中的8执行partition函数，分为两个部分。

2,7,3,5,6,1,8,9,11,14,10,12,19,32,17
因为Position (8)=7>3，所以在8的前半部分(L) 查找第3个元素(k-th)。

(5) 重复(1)~(4) 的过程，直到Position (关键字)=3。

算法描述引用自stack overflow [eladv](http://stackoverflow.com/users/7314/eladv) 与 [dilip-raj-baral](http://stackoverflow.com/users/1175279/dilip-raj-baral)的回答:

```c
Select(A,n,i):

Divide input into ⌈n/5⌉ groups of size 5.

//Partition on median-of-medians

medians = array of each group’s median.

pivot = Select(medians, ⌈n/5⌉, ⌈n/10⌉)

Left Array L and Right Array G = partition(A, pivot)

//Find i-th element in L, pivot, or G

k = |L| + 1

If i = k, return pivot

If i < k, return Select(L, k-1, i)

If i > k, return Select(G, n-k, i-k)"
```

最后祝愚人节快乐，晚安。
