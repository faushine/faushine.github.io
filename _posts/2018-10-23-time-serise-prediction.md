---
title: ARIMA建模实例
date: 2018-10-23 
tags: 
- ARIMA
author: Faushine
layout: post
---

之前写过一篇关于时间序列的文章，下面来说一说怎么把ARIMA算法应用的具体的dataset上，因为我在最近的Data Analytics课程作业里用到了这个算法，我就暂且用这个例子来讲讲。

## 数据集
数据集包括了从2004年1月1日到2008年6月1日13个不同地区每小时的气温观测值，我抽取了地区1的数据来做预测，并以每天的最高气温为研究对象。这一步要做的工作就是整理数据集，生成日期，计算每日最大值。

### 数据集划分
数据被划分为训练集，验证集和测试集，比例如下图所示。
![alt text](/img/in-post/2018-10-23/1.jpg)

## 稳定性
数据集划分好了我们画个图看看他们长什么样子。
![alt text](/img/in-post/2018-10-23/2.jpg)
![alt text](/img/in-post/2018-10-23/3.jpg)
![alt text](/img/in-post/2018-10-23/4.jpg)

很明显training set是不稳定的，它呈现了很明显的季节性，我们计算一下ADF看看：
```python
ts = trainmax['max']
adf = adfuller(ts)
print(adf)
```
Results of ADF Test:
```
(-2.284435824985588, 0.17703687223179454, 13, 971, {'5%': -2.8645211564578115, '10%': -2.568357325328449, '1%': -3.437102493904439}, 6435.36264808484)
```
如何确定该序列能否平稳呢？主要看：

1. 1%、%5、%10不同程度拒绝原假设的统计值和ADF Test result的比较，ADF Test result同时小于1%、5%、10%即说明非常好地拒绝该假设，本数据中，adf结果为-2.284435824985588，大于三个level的统计值。
2. P-value是否非常接近0.而在本数据中，P-value 为0.17703687223179454大于0.01。

也就是说当前的数据还不平稳。

对于不平稳的处理：
1. 对数处理: 对数处理可以减小数据的波动，因此无论第1步检验出序列是否平稳，都最好取一次对数。关于为什么统计、计量学家都喜欢对数的原因，知乎上也有讨论：在统计学中为什么要对变量取对数？
2. 差分: 一般来说，非纯随机的时间序列经一阶差分或者二阶差分之后就会变得平稳。那差分几阶合理呢？我的观点是：在保证ADF检验的p<0.01的情况下，阶数越小越好，否则会带来样本减少、还原序列麻烦、预测困难的问题。

我们做一次差分看看：
```python
diff = trainmax.diff(1)
diff = diff.dropna(axis=0, how='any')
diff.plot(figsize=(15,6))
diff.plot(diff)
```
一次差分之后图像如下所示：
![alt text](/img/in-post/2018-10-23/5.jpg)

同样的用ADF去检查是不是平稳：
![alt text](/img/in-post/2018-10-23/6.jpg)

从上表可以看出来一次差分之后的图像已经平稳了。

## 定阶
AR和MA的阶数可以通过ACF和PACF图像来得到，一次差分之后的ACF及PACF图像如下图所示：
```python
from statsmodels.graphics.tsaplots import plot_acf
from statsmodels.graphics.tsaplots import plot_pacf

pplt.figure()

plot_acf(diff, lags=20)
plot_pacf(diff,lags=20)
pplt.plot()
```
![alt text](/img/in-post/2018-10-23/7.jpg)
![alt text](/img/in-post/2018-10-23/8.jpg)

那么怎么通过这两种图去确定阶数呢？
>The dataset can be used to fit an ARIMA(p,d,0) model if:
> * the ACF is exponentially decaying or sinusoidal;
> * there is a significant spike at lag p in PACF, but none beyond lag p.
> 
>The dataset can be used to fit an ARIMA(0,d,q) >model if:
>* the PACF is exponentially decaying or sinusoidal;
>* there is a significant spike at lag q in ACF, but none beyond lag q.

由上图我们可以看到，PACF图像呈现正弦趋势，PACF图像在lag=1的位置有比较明显的峰值，所以假设 (0,1,1)是可能的阶数组合之一。

另外一种确定阶数的方法是搜索所有可能的组合，分别计算它们的RSS值，RSS比较小的模型会更加收敛。假设p d的范围在0到3之间。
``` python
from statsmodels.tsa.arima_model import ARIMA

def get_safe_RSS(series, fitted_values):
    fitted_values_copy = fitted_values  # original fit is left untouched
    missing_index = list(set(series.index).difference(set(fitted_values_copy.index)))
    if missing_index:
        nan_series = pd.Series(index = pd.to_datetime(missing_index))
        fitted_values_copy = fitted_values_copy.append(nan_series)
        fitted_values_copy.sort_index(inplace = True)
        fitted_values_copy.fillna(method = 'bfill', inplace = True)  # fill holes
        fitted_values_copy.fillna(method = 'ffill', inplace = True)
    return sum((fitted_values_copy - series)**2)

ARIMA_fit_results = {}
Diff=1
for AR in (0,3) :
    for MA in (0,3) :
        model = ARIMA(ts, order = (AR,Diff,MA))
        fit_is_available = False
        results_ARIMA = None
        try:
            results_ARIMA = model.fit(disp = 0, method = 'css')
            fit_is_available = True
        except:
            continue
        if fit_is_available:
            safe_RSS = get_safe_RSS(ts, results_ARIMA.fittedvalues)
            ARIMA_fit_results['%d-%d-%d' % (AR,Diff,MA)]=[safe_RSS,results_ARIMA]

best_ARIMA = min(ARIMA_fit_results, key = ARIMA_fit_results.get)
print(best_ARIMA)
ARIMA_fit_results
```
我们得到如下结果：
![alt text](/img/in-post/2018-10-23/9.jpg)

通过上表我们得到了两组可能的阶数组合：(0,1,1)和(3,1,3)

## 训练模型
下面我们把ARIMA(0,1,1)和ARIMA(3,1,3)分别应用在训练集上：
```python
import warnings
import itertools
import pandas as pd
import numpy as np
import statsmodels.api as sm

mod = sm.tsa.statespace.SARIMAX(trainmax,
                                order=(0, 1, 1),
                                enforce_stationarity=False,
                                enforce_invertibility=False)

results = mod.fit(disp=0)

print(results.summary())
```
ARIMA(0,1,1):
![alt text](/img/in-post/2018-10-23/10.jpg)

ARIMA(3,1,3):
![alt text](/img/in-post/2018-10-23/11.jpg)

## 验证模型
把上一部分训练好的模型应用到验证集中做预测，在不同的模型间进行误差比较。
```python
from sklearn.metrics import mean_squared_error
import warnings
import itertools
import pandas as pd
import numpy as np
import statsmodels.api as sm

mod = sm.tsa.statespace.SARIMAX(trainmax,
                                order=(3, 1, 3),
                                enforce_stationarity=False,
                                enforce_invertibility=False)

results = mod.fit(disp=0)
pred = results.get_prediction(start=pd.to_datetime('2006-09-12'), dynamic=False)
pred_ci = pred.conf_int()

trainshow = trainmax['2006-08-12':]
trainshow['yhat'] = results.fittedvalues

fig, ax = pplt.subplots(figsize=(20,10))
ax.plot(trainshow.index, trainshow['max'], 
       label='Observed temperature')
ax.plot(pred.predicted_mean.index, pred.predicted_mean, 
       label='One-step ahead forecast', alpha=.7)
fig.autofmt_xdate()
ax.fill_between(pred_ci.index,
               pred_ci.iloc[:, 0],
               pred_ci.iloc[:, 1], color='k', alpha=.2)
ax.set_xlabel('Date')
ax.set_ylabel('Max Temperature')
ax.set_title('ARIMA(3,1,3) In sample one-step prediction')
pplt.legend()
```
验证集上的预测结果：
ARIMA(0,1,1):
![alt text](/img/in-post/2018-10-23/12.jpg)

ARIMA(3,1,3):
![alt text](/img/in-post/2018-10-23/13.jpg)

MSE分别如下所示：
![alt text](/img/in-post/2018-10-23/14.jpg)

## 预测
通过前一部分的误差比较，我们选择误差较小的ARIMA(3,1,3)在测试集上进行预测。
```python
from pandas import Series
from matplotlib import pyplot
from statsmodels.tsa.arima_model import ARIMA
from statsmodels.tsa.arima_model import ARIMAResults
from scipy.stats import boxcox
from sklearn.metrics import mean_squared_error
from math import sqrt
from math import exp
from math import log
import numpy
 
# invert box-cox transform
def boxcox_inverse(value, lam):
	if lam == 0:
		return exp(value)
	return exp(log(lam * value + 1) / lam)
 
# load and prepare datasets
dataset = trainmax
X = dataset.values.astype('float32')
history = [x for x in X]
validation = testmax
y = validation.values.astype('float32')
# load model
model_fit = ARIMAResults.load('model.pkl')
lam = numpy.load('model_lambda.npy')
# make first prediction
predictions = list()
yhat = model_fit.forecast()[0]
yhat = boxcox_inverse(yhat, lam)
predictions.append(yhat)
history.append(y[0])
print('>Predicted=%.3f, Expected=%3.f' % (yhat, y[0]))
# rolling forecasts
for i in range(1, len(y)):
	# transform
	transformed, lam = boxcox(history)
	if lam < -5:
		transformed, lam = history, 1
	# predict
	model = ARIMA(transformed, order=(0,1,1))
	model_fit = model.fit(disp=0)
	yhat = model_fit.forecast()[0]
	# invert transformed prediction
	yhat = boxcox_inverse(yhat, lam)
	predictions.append(yhat)
	# observation
	obs = y[i]
	history.append(obs)
	print('>Predicted=%.3f, Expected=%3.f' % (yhat, obs))
# report performance
mse = mean_squared_error(y, predictions)
rmse = sqrt(mse)
print('MSE: %.3f' % mse)
print('RMSE: %.3f' % rmse)
```
预测结果如下图所示：
![alt text](/img/in-post/2018-10-23/15.jpg)
![alt text](/img/in-post/2018-10-23/16.jpg)


如果你有兴趣的话可以访问我的[git仓库](https://github.com/faushine/ECE-9063/tree/master/1
)，包含了本次课程作业的源码及数据文件。


最后祝各位1024节快乐，我的愿望是

![alt text](/img/in-post/2018-10-23/16.png)


不对，是再也不住Airbnb。
