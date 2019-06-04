---
title: 用Matplotlib 画直方图
date: 2018-07-11 18:35:03
tags: 
- Python
author: Faushine
layout: post
---
最近有个需求要对文件的内容做统计，要求能够比较直观的看到数据分布情况。Matplotlib是目前Python2D绘图领域使用最广泛的套件，它能让使用者很轻松地将数据图形化，并且提供多样化的输出格式，于是决定用它来做这个事情。

首先假设数据集是以下四个list：
```python
diff_value_1,diff_value_2,diff_value_3,diff_value_4﻿​ 
```
```python
plt.figure(figsize=(12,6))﻿​ #指定图的大小  
(counts,bins,patch) = plt.hist(\[diff\_value\_1,diff\_value\_2,diff\_value\_3,diff\_value\_4\],bins=20,color=\['green','blue','red','yellow'\], alpha=0.5,label=('02-03', '03-04','04-05','05-06'))  
#设置直方图的属性：data，bins(直方图的条数)，color，alpha(透明度)，label
#等号左边的元组是统计结果，counts就是分类之后的统计值 
 
 plt.xlim(0,1) #X轴的最大区间
 ax = plt.gca() 
 ax.yaxis.set\_major\_locator(MultipleLocator(type_num)) #y轴的主刻度
 ax.yaxis.set\_minor\_locator(MultipleLocator(type_num/2)) #y轴的副刻度
 ax.xaxis.set\_major\_locator(MultipleLocator(0.1)) #x轴的主刻度
 ax.xaxis.set\_minor\_locator(MultipleLocator(0.05)) #x轴的副刻度  
 plt.xlabel('Different value exist(%)') #设置x轴的标签
 plt.ylabel('Count') #设置y轴的标签
 plt.title(image) #设置表的名字
 plt.grid() #设置网格显示
 plt.legend() #设置直方图每种颜色的图例
 plt.annotate('text', xy=(0.2, 0), xytext=(0.2, type_num), arrowprops={'arrowstyle': '->'}) 
 #设置标注，通过 arrowprops 参数可以设置标注文字指向点的箭头，接收参数为一个字典。包括 arrowstyle，connectionstyle 等等。arrowstyle 内置了很多种样式，'->' 表示一个箭头。
 plt.savefig(image+'.png') #保存图表
 plt.show() #显示图表
```
效果图：
![alt text](/img/in-post/2018-07-11/MSGBLFND 02 - 06 Beta.png "50万条数据统计")

![alt text](/img/in-post/2018-07-11/MSHYBFND 02 - 06 Beta.png "1000条数据统计")

[源代码下载](/img/in-post/2018-07-11/source.7z)