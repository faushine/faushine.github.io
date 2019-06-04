---
title: 时间序列及其预测模型
date: 2018-09-30 
tags: 
- ARIMA
mathjax: true
author: Faushine
layout: post
---

课堂笔记

***
教材：
[Forecasting: Principles and Practice](https://otexts.org/fpp2)

三个基本原则
- 过去数据中的随机波动可以忽略掉，即异常数据
- 真实的数据一定会被用于建模和计算
- 我们假设：环境变化的方式不会随着时间改变

那么我们要预测什么呢？
- 每个产品的销量还是一组产品的销量
- 每个销售点的销量，一组销售点的销量，还是总体的销量
- 每小时，每天，每周还是每年的销量
必须明确的一点是，预测的精确度是随着多样性的增加而降低的，也就是说预测个体的值相对整体来说误差会更大。当我们选取预测对象的时候就要留意对象的范围，太精确会增大误差。

下面进入正题
## 1. 时间序列组成模式
1. 趋势Tend ：比如线性趋势，先增加后降低的整体趋势 
2. 季节性Seasonal ：以时间为固定周期，呈现循环的特性 
3. 周期性Cyclic：在以不固定周期不断震荡，通常周期性至少持续2年 

趋势很好理解，如果图像呈现明显的上升或者下降的趋势，那么就是trend。季节性可以理解成图像会依照某种固定的时间周期波动，而且波动在每个周期内都是相似的。周期性比较confusing，我的理解是图像周期性震荡，而且周期不固定持续两年以上，震荡随机就是cyclic。（好像跟定义说的差不多）

***

## 2. 时间序列分解
时间序列可以被分解成上述几种模式，分解方式有两种：`Additive` 和 `Multiplicative`，即相加和相乘。
- 相加模型
  $Y_{t} = S_{t} + T_{t} + E_{t}$
- 相乘模型
  $Y_{t} = S_{t} \times T_{t} \times E_{t}$

$Y_{t} \to$数据点 data
$S_{t} \to$季节性模式部分 seasonal component 
$T_{t} \to$趋势性和周期性模式部分 trend-cycle component
$E_{t} \to$误差部分 error

![alt text](/img/in-post/2018-10-01/exp1-1.png "图2.1 红线表示趋势周期性，灰线是原始数据.")


![alt text](/img/in-post/2018-10-01/exp1-2.png "图2.2 加法模型STL方法分解之后的数据图像，右边的灰色标度是以reminder为基准，我们可以看出残差部分的范围较小。")


什么时候使用这两种模型？
1. 如果时间的变化对季节性和趋势周期性影响比较小，这个时候我们就可以去除季节性。比如预测失业数据的时候，每年毕业季失业人数会增多，这个变化是具有季节性的，但是我们还得考虑的是经济环境的变化，如果经济衰退的话失业人口也会增多，而这个变化是没有季节性的。大多数的经济学家在研究这个问题的时候更倾向于考虑非季节性的变化，所以常常需要去除季节性的数据(seasonal adjusted data)：
   - 对于加法模型：
   $Y_{t} - S_{t}$ 
   
   - 对于乘法模型:
   $Y_{t} \div S_{t}$ 

   ![alt text](/img/in-post/2018-10-01/exp2.png "图2.3 蓝色部分是去除季节性之后的数据图像，灰色是原始数据图像")
   

2. 季节性和趋势周期性的波动随时间成比例，适用于乘法模式。而乘法模式可以通过log变换转成加法模式：
   $log{Y_{t}} = log{S_{t}} + log{T_{t}} + log{E_{t}}$

### 2.1 移动平均 Moving averages
Moving average(m-MA) of order $m$ :

$\hat{T_{t}} = \frac{1}{m}\sum_{j=-k}^{k}y_{t+j}$
$m = 2k+1$

由上式我们可以看出：
- 用 $t$ 前后k个数据点的平均值来表示$t$。
- 最前 $k$ 个和最后 $k$ 个数据点为 $null$
- $m$ 越大曲线越平滑
- 移动平均可以反映长期的趋势和周期
  
![alt text](/img/in-post/2018-10-01/exp3.png "表2.1 Annual electricity sales to residential customers in South Australia. 1989–2008.")

$5-MA$ 的第一个值表示1989-1993的平均值，第二个值表示1990-1994的平均值，以此类推

### 2.2 移动平均的移动平均
当 $m$ 为奇数的时观测点前后有 $k$ 个值，当 $m$ 为偶数的时候就得再做一次移动平均，我们把得到的结果称为centered moving average of order m。
- 偶*偶-MA
- 奇*奇-MA
![alt text](/img/in-post/2018-10-01/exp4.png "表2.2 A moving average of order 4 applied to the quarterly beer data, followed by a moving average of order 2.")

$2\times4-MA$ 表示先经过一次 $4-MA$，再经过一次 $2-MA$ 之后的结果。

$4-MA$:
451.25=(443+410+420+532)/4 
448.75=(410+420+532+433)/4
其余项不赘述

$2\times4-MA$:
450.00=(451.25+448.75)/2

对于具有季节性的数据：
- 如果周期是偶数，用 $2\times m-MA$。比如 monthly 用$2\times12-MA$
- 如果周期是奇数，用$m-MA$。比如 weekly 用$7-MA$

### 2.3 加权的移动平均 m-MA
定义：
$\hat{T_{t}} = \frac{1}{m}\sum_{j=-k}^{k}a_{j}y_{t+j}$  
且$k=(m-1)/2$, $[a_{-k},...,a_{k}]$ 表示权值.

由上式可知，加权移动平均就是在计算的时候给每个数值加权，比如上述例子中的 $2\times4-MA$ 可以看成权值$[\frac{1}{8},\frac{1}{4},\frac{1}{4},\frac{1}{4},\frac{1}{8}]$的 $5-MA$。$m-MA$ 是权值都为$\frac{1}{m}$的一种特殊情况。

***

## 3. 经典分解
假设：seasonal component是连续的。

### 3.1 相加的分解
步骤如下：
1. 季节性周期 $m$ 是偶数时，使用 $2\times m-MA$来估计trend-cyclic $y−\hat{T}$，季节性周期 $m$ 是奇数时，使用 $m-MA$ 来估计trend-cyclic $\hat{T}$
2. 计算去趋势序列 $y_{t}−\hat{T}$
3. 计算每个周期的去趋势平均值 $S_{t}$，用该值来表示当前周期的值。比如计算3月的 $S_{t}$，就得算所有三月的去趋势序列 $y_{t}−\hat{T}$ 的平均值。
4. 计算剩余部分 $\hat{R_{t}} = y_{t}-\hat{T_{t}}-\hat{S_{t}}$

### 3.2 相乘的分解
1. 同相加分解
2. 计算去趋势序列 $y_{t}/{\hat{T_{t}}}$
3. 同相加分解
4. 计算 $\hat{R_{t}} = y_{t}/(\hat{T_{t}}\hat{S_{t}})$
  
### 3.3 经典分解的不足
1. 使用moving average时会使数据的前面和后面的数值缺失，变为null。
2. 季节性的趋势可能会随时间变化，而经典中使用了平均值，忽略了这一变化。
3. 求平均值时，容易受到异常值的影响。

***

## 4. 时间序列预测模型

AR, MA, ARMA, 以及 ARIMA。

### 4.1 平稳性 Stationarity
平稳性，即时间序列的行为不随时间而改变。所以具有趋势的（trend）或者季节性的（seasonal）序列都不是平稳的，一个平稳的时间序列在图形上往往表现出一种围绕其均值不断波动的过程，如白噪声。
怎么判断时间序列是不是平稳的呢？
1. 通过趋势图判断
2. 通过自相关函数ACF判断

#### (1) 自相关函数 ACF(Autocorrelation Function)

定义：

$r_{k}=\frac{\sum_{T}^{t=k+1}(y_{t}-\hat{y})(y_{t-k}-\hat{y})}{\sum_{t=1}^{T}(y_{t}-\hat{y})^2}$
`偏自相关函数`： 延迟为 $k$ 时，度量相距 $k$ 个时间间隔的序列值之间的相关性。如 $r_{1}$ 描述的是 $y_{t}$ 和 $y_{t-1}$ 之间的相关性。

平稳序列通常具有短期的相关性，其ACF图像会快速衰减到0，而对于非平稳序列来说，其ACF图像会缓慢衰减，另外其 $r_{1}$ 的值往往是个正数。如下图所示：

![alt text](/img/in-post/2018-10-01/acfs.png "图4.1 The ACF of the Google stock price (left) and of the daily changes in Google stock price (right).")


下面思考一个问题：如果 $y_{t}$ 和 $y_{t-1}$ 是相关的，而 $y_{t-1}$ 和 $y_{t-2}$ 一定也是相关的，那么 $y_{t}$ 和 $y_{t-2}$ 是不是也是相关的呢？我们可以用偏自相关函数PACF（partial autocorrelation function）去解决这问题。

#### (2) 偏自相关函数 ACF(Autocorrelation Function)

`偏自相关函数`： 延迟为$k$时，度量相距$k$个时间间隔的序列值之间的相关性，同时去除了间隔之间的值的影响。由此可见，ACF和RACF的区别就在于是否剔除中间$k-1$个随机变量的干扰。

#### (3) 差分
因为实际的时间序列数据往往是不平稳的，这会导致出现“虚假回归”，也就是说两个没有任何因果关系的变量却有很高的相关性情况，所以我们需要把不平稳的序列变成平稳序列，也就是差分。差分的次数取决于图像的变化情况，直到图像变成平稳型为止。

### 4.2 自回归模型(AR) Autoregressive models
自回归模型（Autoregressive model，简称AR模型），是统计上一种处理时间序列的方法，用同一变数例如 $x$ 的之前各期，亦即 $x_{1}$ 至 $x_{t-1}$ 来预测本期 $x_{t}$ 的表现，并假设它们为——线性关系。因为这是从回归分析中的线性回归发展而来，只是不用 $x$ 预测 $y$，而是用 $x$ 预测 $x$自己，所以叫做自回归。

Model of order $p$ or AR(p) model:

$y_{t}=c+\phi_{1}y_{t-1}+\phi_{2}y_{t-2}+...+\phi_{p}y_{t-p}+\varepsilon_{t}$
$p$ 被称作AR模型的阶数。

对于AR(1)模型：
当 $\phi_{1} = 0$ 时，$y_{t}$ 相当于白噪声；
当 $\phi_{1} = 1$ 且 $c=0$ 时，$y_{t}$ 相当于随机游走；
当 $\phi_{1} = 1$ 且 $c\neq0$ 时，$y_{t}$ 相当于带漂移项的随机游走；
当 $\phi_{1} < 0$ 时，$y_{t}$ 在正负之间震荡。

我们通常把AR模型限定在平稳数据上，对于其他参数也有如下限制条件：
对于AR(1)模型： $-1 < \phi_{1} < 1$;
对于AR(2)模型： $-1 < \phi_{2} < 1$, $\phi_{1} + \phi_{2} < 1$, $\phi_{2}-\phi_{1}<1$;

当$p\ge3$时，限制条件会变得更加复杂，R会帮你处理。

#### 自回归模型定阶

当ACF拖尾，PACF在p阶截尾时，可以用AR(p)模型；

例如，对于模型 $X_{t}=0.9X_{t-1}-0.3X_{t-2}+\varepsilon_{t}$，它的 ACF 和 PACF 如下。

![alt text](/img/in-post/2018-10-01/pacf.png )

我们可以看出自相关系数呈现一定的周期性，故判定为拖尾；偏自相关系数 2 步后截尾。因此，我们可以尝试使用 AR(2) 模型来建模。

### 4.3 移动平均模型(MA) Moving Average Model
移动平均模型（简记为：MA模型）是一个常见的对单一变量时间序列进行建模的方法。
q 阶移动平均模型通常简记为MA(q)：

$y_{t} = c+\varepsilon_{t}+\theta_{1}\varepsilon_{t-1}+\theta_{2}\varepsilon_{t-2}+...+\theta_{q}\varepsilon_{t-q}$,

其中 $c$ 是序列的均值，$\theta_{1},...,\theta_{q}$ 是参数，$\varepsilon_{t}, \varepsilon_{t-1},...,\varepsilon_{t-q}$都是白噪声。
MA模型不能和移动平均值混淆，MA模型是用于预测未来的值，而移动平均值是根据过往的数据度量序列的趋势性。

#### 移动平均模型定阶

当PACF拖尾，ACF在 $q$ 阶截尾时，可以用MA（q）模型；

### 4.4 ARMA模型
是AR模型和MA模型的结合，定义如下：

$Y_{t} = c+\varepsilon_{t}+\sum_{i=1}^{p}\varphi_{i}Y_{t-i}+\sum_{j=1}^{q}\theta_{i}\varepsilon_{t-j}$

#### ARMA模型定阶

当ACF在q阶段拖尾，PACF在p阶截尾时，可以用ARMA(p,q)模型

### 4.5 ARIMA模型
ARIMA模型（英语：Autoregressive Integrated Moving Average model），差分整合移动平均自回归模型，又称整合移动平均自回归模型（移动也可称作滑动），时间序列预测分析方法之一。 ARIMA(p，d，q)中，AR是"自回归"，p为自回归项数；MA为"滑动平均"，q为滑动平均项数，d为使之成为平稳序列所做的差分次数（阶数）。 AR模型和MA模型是d为0时的特殊情况。ARIMA(p，d，q)模型是ARMA(p，q)模型的扩展。

ARIMA(p，d，q)模型可以表示为：

$y_{t}^{,} = c+\phi_{1}y_{t-1}^{,}+...+\phi_{p}y_{t-p}^{,}+\theta_{1}\varepsilon_{t-1}+...+\theta_{q}\varepsilon_{t-q}+\varepsilon_{t}$

#### ARIMA模型定阶

差分次数为d，p和q根据ACF和PACF的图像决定。

#### ARIMA建模实例

[Step-by-Step Graphic Guide to Forecasting through ARIMA Modeling using R – Manufacturing Case Study Example (Part 4)](http://ucanalytics.com/blogs/step-by-step-graphic-guide-to-forecasting-through-arima-modeling-in-r-manufacturing-case-study-example/)
 
***

## 5 误差分析
此时建立的未必是最优的。一个好的模型通常要求残差序列方差较小，同时模型也相对简单，即要求阶数较低。因此我们需要一些准则来比较不同阶数的模型之间的优劣，从而确定最合适的阶数。

待续。。。。




