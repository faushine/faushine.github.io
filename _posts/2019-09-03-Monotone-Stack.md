---
layout: post
title: "Monotone Stack"
date:   2019-09-03
author: Faushine
tags: 
- Leetcode
---

## 什么是单调栈

单调栈就是一种栈内元素保持单调增或者单调减的栈，我们只在栈顶操作元素。单调栈的性质可以总结为：

1. 单调栈里的元素具有单调性

2. 元素加入栈前，会在栈顶把破坏栈单调性的元素都删除
   
## 单调栈如何维护其单调性

如果stack单调递增，栈顶元素大于等于新元素a：

1. 不断 pop() 栈顶元素。

2. 若 peek() 元素小于a，则 push(a)。

因此，单调栈的维护是 O(n) 级的时间复杂度，因为所有元素只会进入栈一次，并且出栈后再也不会进栈了。

## 单调栈的用法

下面分享关于单调栈的两个题：[Leetcode 84](https://leetcode.com/problems/largest-rectangle-in-histogram/) 和 腾讯秋招笔试第四题：

---

## Leetcode 84: Largest Rectangle in Histogram

Given n non-negative integers representing the histogram's bar height where the width of each bar is 1, find the area of largest rectangle in the histogram.

![alt text](/img/in-post/2019-09-03/histogram.png)

Above is a histogram where width of each bar is 1, given `height = [2,1,5,6,2,3]`.

![alt text](/img/in-post/2019-09-03/histogram_area.png)

The largest rectangle is shown in the shaded area, which has `area = 10` unit.

---

对于这个题首先想到的思路就是暴力破解，两个for循环对每一个元素求最大矩形面积，这样做的时间复杂度是 O(n^2) 而且不能通过 leetcode 的测试用例。我们可以使用一个单调递增栈让时间复杂度降到 O(n)。

由题目可知矩形的大小取决于最小的元素以及宽度，在一个单调递增栈中我们可以确定先pop的元素一定可以覆盖后pop的元素，所以我们比较栈的元素的最大矩形就能得到所有元素的最大矩形。这样说可能比较抽象，我们举个栗子：

<img src="/img/in-post/2019-09-03/1.jpg" height="300px" width="300px">

栈只保存元素的index，每遍历一次我们会对栈进行两种操作：压栈或者计算结果

- 什么时候压栈：当栈顶元素为-1或者`data[栈顶元素]`小于当前元素时。

- 什么时候计算结果：当`data[栈顶元素]`大于等于当前元素时，弹出栈顶元素，计算该元素的结果。

- 结果怎么计算：长度的下标为`pop()`，宽度的下标为`当前元素的index - peek() - 1`。从(i-1)到peek()这个范围的矩形高度都小于等于pop()元素对应的高度，也就是说这个范围的矩形能够被该元素覆盖。

初始化：

```java
stack.push(-1);
data.add(0);
```

将`-1`压到栈底是为了抵消计算第一个元素时的宽度-1，如果不加的话无法计算栈底的元素。数组的末尾添加元素0是为了标志数组结尾，如果不加的话就无法计算数组的最后一个元素。

(1) `i = 0`, 栈顶元素为`-1`，将 `0` 压到栈中。

\|0\|

\|-1\|


(2) `i = 1`, 因为 `data[stack.peek()]=2>data[1]=1`，所以`0`出栈，计算矩形面积；`data[0] * (1-(stack.peek()=-1)-1)=2*(1-(-1)-1)=2`。这样我们就得到阴影部分的面积为2。此时栈已经空了，所以不需要再计算面积，将`1`压入栈中。

\|1\|

\|-1\|


<img src="/img/in-post/2019-09-03/2.jpg" height="300px" width="300px">


(3) `i = 2`，因为`data[stack.peek()]=data[1]=1<data[2]=5`，所以把`2`压入栈中。同理 `i=3` 时，把`3`压入栈中。

\|3\|

\|2\|

\|1\|

\|-1\|

(4) `i = 4`，因为`data[stack.peek()]=data[3]=6 > data[4]=2`，所以`3`出栈，计算矩形面积：`data[0] * (i-stack.peek()-1)=data[3] * (4-(2)-1)=6 * 1=6`，于是得到阴影部分的矩形面积为6。

\|2\|

\|1\|

\|-1\|


<img src="/img/in-post/2019-09-03/3.jpg" height="300px" width="300px">


因为`data[stack.peek()]=data[2]=5 > data[i]=4` 所以继续计算矩形面积，`2`出栈，矩形面积为：`data[2] * (i-stack.peek()-1)=data[2] * (4-(1)-1)=5 * 2=10`，阴影部分的矩形面积为10。

\|1\|

\|-1\|


<img src="/img/in-post/2019-09-03/4.jpg" height="300px" width="300px">


此时`data[stack.peek()]=data[1]=1 < data[i]=4`，所以停止计算，将`4`压入栈中。

\|4\|

\|1\|

\|-1\|

(5) `i = 5`，因为`data[stack.peek()]=data[4]=2 < data[i]=3`，所以将`5`压入栈中。

\|5\|

\|4\|

\|1\|

\|-1\|

(6) `i = 6`，因为`data[stack.peek()]=data[5]=3 > data[i]=0`，所以`5`出栈，计算矩形面积：`data[5] * (i-stack.peek()-1)=data[5] * (6-(4)-1)=3 * 1=3`，阴影部分的矩形面积为3。

\|4\|

\|1\|

\|-1\|

<img src="/img/in-post/2019-09-03/5.jpg" height="300px" width="300px">


因为`data[6]=0`，所以栈内除-1外的所有的元素都比它大，我们可以不断用栈顶元素计算矩形面积：

`4` 出栈，此时阴影部分矩形面积为`data[4]*(6-(1)-1)=2*4=8`。

\|1\|

\|-1\|

<img src="/img/in-post/2019-09-03/6.jpg" height="300px" width="300px">

`1` 出栈，此时阴影部分矩形面积为`data[1]*(6-(-1)-1)=1*6=6`。

\|-1\|

<img src="/img/in-post/2019-09-03/7.jpg" height="300px" width="300px">

(7) 循环结束，返回最大的矩形面积`10`。

   
附代码：

```java
import java.util.Stack;

public class LargestRectangleinHistogram {

  public int largestRectangleArea(int[] heights) {
    if (heights == null || heights.length == 0) {
      return 0;
    }
    Stack<Integer> stack = new Stack<>();
    int max = 0;
    stack.push(-1);
    for (int i = 0; i <= heights.length; i++) {
      int curt = -1;
      if (i < heights.length) {
        curt = heights[i];
      }
      while (stack.peek() != -1 && heights[stack.peek()] >= curt) {
        int h = heights[stack.pop()];
        int w = i - stack.peek() - 1;
        max = Math.max(max, h * w);
      }
      stack.push(i);
    }
    return max;
  }
}

```
---
## 腾讯秋招笔试题

![alt text](/img/in-post/2019-09-03/tencent.png)

<!-- <img src="/img/in-post/2019-09-03/tencent.jpg" height="300px" width="300px"> -->


![alt text](/img/in-post/2019-09-03/tencent1.png)
<!-- <img src="/img/in-post/2019-09-03/tencent1.jpg" height="300px" width="300px"> -->

---

我们的目标是找到一个区间，这个区间的最小值乘以区间和最大。这个题和上面那个题就很相似，一个是找区间内面积最大，一个找乘积最大。我们知道找一个区间的最值可以用单调栈来做，区间和可以用前缀和来解决。 也就是说问题可以转化成对于一个元素，找到以该元素为最小值的连续区间，并计算区间和乘以该元素的值，返回一个最大值。解题步骤如下：

首先回答三个问题：

- 什么时候压栈：当栈顶元素为-1或者`data[栈顶元素]`小于当前元素时。

- 什么时候计算结果：当`data[栈顶元素]`大于等于当前元素时，弹出栈顶元素，计算该元素的结果。

- 结果怎么计算：栈顶弹出元素下标为`pop()`，区间和为`prefix[当前元素index]-prefix[peek()+1]`。`prefix[i]`表示前i-1个元素的和，`prefix[peek()+1]`表示比该元素小的元素的和，因为peek()也比该元素小，要计算peek(),所以+1。因此用`前i-1个元素的和`减去`比栈顶元素小的元素和`就能得到以栈顶元素为最小值的区间，并计算结果。


初始化：

```java
prefix[];// 计算前缀和
data.add(0);
```

![alt text](/img/in-post/2019-09-03/01.jpg)
<!-- <img src="/img/in-post/2019-09-03/01.jpg" height="500px" width="100px"> -->

i 的前缀和就是前 i-1 个元素的和，注意不包括元素 i，通过前缀和可以得到某一区间内元素的和。和上一题的套路一样，我们在数组的结尾新增一个元素0，用于标志数组的结束。

(1) `i = 0`，data[0] = 7，栈顶元素为-1，不计算结果直接压栈。
   
\|0\|

\|-1\|

(2) `i = 1`，data[1] = 2，栈顶元素0对应data元素7大于2，此时`0出栈`，计算结果：`data[0]*(prefix[1]-prefix[-1+1]) = 7`。
   
\|-1\|

(3) `i = 2`，data[2] = 4，栈顶元素为-1，不计算结果直接压栈。
   
\|2\|

\|-1\|

(4) `i = 3`，data[3] = 6，栈顶元素为2对应data元素4小于6，直接压栈。
   
\|3\|

\|2\|

\|-1\|

(5) `i = 4`，data[4] = 5，栈顶元素3对应data元素6大于5，此时`3出栈`，计算结果：`data[4]*(prefix[4]-prefix[2+1]) = 5*(19-13)=30`。因为现在栈顶元素2对应的data元素4小于5，所以停止计算，将4压栈。
   
\|4\|

\|2\|

\|-1\|

(6) `i = 5`，data[5] = 0，到了数组的末尾，栈内除-1以为的所有元素对应值都大于0，所以弹出栈顶元素并计算：

`4出栈`，结果为：4*(24-9)=60;

`2出栈`，结果为：2*(24-0)=48。

综上，我们找到区间和与最小值的乘积最大值为60。

附代码：

```java
import java.util.Stack;

public class TermReview {

  public static int getHighestEvaluation(int[] scores) {
    int n = scores.length;
    int[] prefix = new int[n + 1];
    prefix[0] = 0;
    for (int i = 1; i <= n; i++) {
      prefix[i] = prefix[i - 1] + scores[i-1];
    }
    Stack<Integer> stack = new Stack<>();
    stack.push(-1);
    int max = 0;
    for (int i = 0; i <= n; i++) {
      int next = (i == n) ? 0 : scores[i];
      while (stack.peek()!=-1 && scores[stack.peek()] >= next) {
        int index = stack.pop();
        int curt = scores[index];
        int result = curt * (prefix[i] - prefix[stack.peek() + 1]);
        max = Math.max(max, result);
      }
      stack.push(i);
    }
    return max;
  }

}
```

---

许愿一个offer