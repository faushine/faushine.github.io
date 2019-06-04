---
title: 最长子串问题
date: 2018-02-26 
tags: 
- DP
- Algorithm
mathjax: true
author: Faushine
layout: post
---

> Given a string, find the length of the **longest substring** without repeating characters. 

> **Examples:**
Given ``"abcabcbb"``, the answer is ``"abc"``, which the length is 3.
Given ``"bbbbb"``, the answer is ``"b"``, with the length of 1.
Given ``"pwwkew"``, the answer is ``"wke"``, with the length of 3. Note that the answer must be a **substring**, ``"pwke"`` is a subsequence and not a substring.


## 方法一：穷举法  ($O$($n^2$)$)$
枚举所有不重复子串可能出现的情况，记录子串长度的最大值。
```java
public int lengthOfLongestSubstring(String s) {
        int rst=0;
        int j;
        char[] master = s.toCharArray();
        Map<Character,Integer> subString = new HashMap<>();
        for (int i = 0;i<master.length;i++){
            subString.clear();
            subString.put(master[i],i);
            for (j = i+1;j<master.length;j++){
                if (subString.get(master[j])!=null){
                    break;
                }
                subString.put(master[j],j);
            }
            rst = Math.max(j-i,rst);
        }
        return rst;
    }
```
## 方法二：动态规划  ($O$($n$)$)$
动态规划(Dynamic programming)是一种将复杂问题分解为简单子问题的编程思想，通过对子问题求解从而得到最优解，跟分治法不同的是动态规划会把子问题的结果存储起来，而不用求解每个子问题时都要计算一遍。
最长子串问题可以描述为：

 - 当前的字符和前面的字符没有重复，那么到当前字符的最长无重复子串就应该在上次求得的串长度+1；
 
 - 如果当前的字符有冲突，那么有两种情况需要分析。第一，如果和它冲突的字符在当前最长无重复子串的起始位置前面，那么很明显，对计算当前的串没有影响，还是在上次基础上+1；如果冲突字符在当前的串开始位置之后，那么就从后一个位置计算无重复子串的长度。



求解动态规划问题的关键是定义**状态**和**状态转移方程**，由上文``状态``可以定义为：


给定一个字符串$s$[$x_1$,$x_2$,$...$, $x_N$]，长度为$N$。
设$L$[$i$]为：字符串中以第$i$项结尾的不重复子串长度. $i$<$N$
求$L$[$1$]$,L[2$]$,$...$,L$[$N$]中的最大值。

``状态转移方程``可以定义为：

令 $L$[$i$] $=$ $s$ [$j$...$i$] $=$ $i$-$j$+$1$

 1. 如果 $s$[$i$+$1$]与$s$[$j$]$,$...$,$ $s$[$i$]$ 中的字符不重复。
L$[$i$+$1$] $=$ $L$[$i$]$ + $1

 2. 如果$s$[$i$+$1$] 与 $s$[$j$]$,$...$, s$[$i$]$ 中的字符重复，设重复字符的位置为k$
则 $j$ $=$ $m$a$x$ ($j$,$k$), 即可得 $L$[$i$+$1$] $=$  $s$[$j$...$,$ $i$+$1$] $=$ ($i$+$1$)$ - j$ $+$ $1$
判断字符是否与前文重复可以通过hashMap来实现。

代码实现如下：
```java
public int lengthOfLongestSubstring(String s) {
        if (s.length()==0) return 0;
        HashMap<Character, Integer> map = new HashMap<Character, Integer>();
        int max=0;
        for (int i=0, j=0; i<s.length(); ++i){
            if (map.containsKey(s.charAt(i))){
                j = Math.max(j,map.get(s.charAt(i))+1);
            }
            map.put(s.charAt(i),i);
            max = Math.max(max,i-j+1);
        }
        return max;
    }
```




