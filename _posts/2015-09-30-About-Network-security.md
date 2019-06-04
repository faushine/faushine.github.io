---
layout: post
title: "阅读X-Force Report有感"
date:   2015-09-30
author: Faushine
tags: 
- Reading
---
## 引言

  IBM X-Force 2013 Mid-Year Trend and Risk Report 是一份对最新安全威胁和趋势进行深入分析的报告，其中包括了对漏洞，攻击和主动攻击，病毒和其他恶意软件，垃圾邮件，网络钓鱼和恶意Web内容等对象的研究。  众所周知，随着计算机网络的普及，利用安全漏洞进行的攻击越来越频繁，与此同时也有越来越多的专业人士致力于解决这些使计算机受到威胁的问题，这份报告的目的就是对大量新型漏洞的介绍来使企业和机构的安全管理更加清晰，文中所讨论的方法也能够使那些对安全感兴趣的非专业人士去加强他们个人账户的管理。

 报告内容概览：

> * Targeted attacks and data breaches

> * Social and mobile

> * Vulnerabilities and exploitation

> * Web trends, spam and phishing

> * Security practice

## 正文

sql注入一直是最普遍的泄露典范，也是得到数据库中的记录最直接的方法，同时其自动化脚本可以对潜在的目标进行广泛的扫描，并且在普通应用程序里面运行那些已知的sql注入漏洞。然而攻击者们不仅会对网站的表单进行sql注入，甚至对于安卓用户还会在其手机app里注入恶意代码。在安卓系统中，各个 app 都把支付平台用 sdk 接入到自己的程序里，因此在同一进程下，即便 sdk 不是攻击者写的代码，他也有一万种方法知道那些输入框里输入了什么。这类攻击比网络钓鱼更难识别，攻击者并不需要绘制输入框，因此如果不在独立进程中做支付，那么结果一定很糟糕。

另一个值得一提的例子是针对社交媒体账号的攻击，攻击者们窃取名人的账户去发布恶意的消息或者虚假广告，更有甚者通过该账号对其好友实施诈骗。社交网络的多元化导致了鱼龙混杂的状况，那些窃取了用户账号的攻击者们创建了黑市交易平台，用于买卖用户账号，一些账号属于证书被攻破的人，而另一些则是创建并设计成可信账户的样子用于产品营销（如点击‘喜欢’）。如果网站的设计中有过滤机制必然会减少僵尸账户的数量，从而使用户不需要去甄别那些僵尸账号。

Daniel J. Bernstein在他回顾qmail十周年的论文中写道：

> ”I see a huge amount of money and effort being invested in security, and I have become convinced that most of that money and effort is being wasted. Most “security” efforts are designed to stop yesterday’s attacks but fail completely to stop tomorrow’s attacks and are of no use in building invulnerable software. ” 

尽管Prof.Bernstein对于安全表现得如此之保守，但是必须提到的是他所开发的qmail却是安全邮件的典范之作，作者曾经悬赏500美金来找出qmail的安全漏洞，但是直到2016年都没有人来领取这份奖金（qmail作为Linux下面主流的邮件系统内核，大量著名的商业邮件系统都是基于它而开发的，比如Hotmail）。
对于如何减少软件的安全漏洞他在后文给出了三个回答 ：

> * Answer 1: eliminating bugs

> * Answer 2: eliminating code

> * Answer 3 eliminating trust code

从Prof. Bernstein 的建议中我们可以看到，如果一开始的设计就做好，或者说在进行安全设计的时候能够考虑尽可能多的情况，精简代码，遵循最小特权原则，替用户把每一个细节都考虑清楚，那么一个优良而安全的程序便孕育而生了。

漏洞和攻击手段层出不穷，即便是大型的互联网公司已经应接不暇，那么对于普通用户来说为了个人信息的安全去尝试着用这些复杂的方法去管理计算机并不是常见的现象。在某种程度上安全和隐私是少数人的特权，那些人包括不计时间成本去管理每一个细节的人，他们关注每一步操作的方方面面，比如下载软件时验证其完整性，比如每一个账号使用不同的密码并保证每个密码超过八位含有大小写数字；以及那些不计经济成本的人，每一项通讯设备都是个人定制并且雇佣专人定期检查，所有与外界连接的接口都被仔细照看。然而大部分的人都不属于这两类，我想只有更简洁更完备的程序和安全策略才能使普通用户也能参与其中。

## 参考

［1］	[IBM X-Force 2013 Mid-Year Trend and Risk Report](/img/in-post/IBM_XForce_2013_2013report.pdf)

［2］	[Daniel J. Bernstein，Some thoughts on security after ten years of qmail 1.0，Department of Mathematics, Statistics, and Computer Science(M/C 249).]( http://cr.yp.to/qmail/qmailsec-20071101.pdf)
