# 开发思路

## Input Features
### 关于宏观环境描述的特征
```
这部分主要是让机器感知到当前环境的冷暖，
其实对于上升趋势没有特别明确的数学定义都是人们凭"感觉"发明出来的，
在新的趋势形成之前是无法判断上一个趋势是否已经结束了，
最多只是判断已经刚刚经历了一个什么趋势。但是很难说自己处于一个什么趋势之中
包括什么是高位和低位，其实连续上涨天数的数量就可以表示 
统计比率为了避免出现数字为零（没有方向性） 尽量选择奇数，
因为KDJ用的参数是【9，3，3】 考虑通常9天的长度已经包含了以涨跌周期所以短期
所以参数选择 9 和 21 日

```
* 最近连续上涨天数数量 值域 0-N  通常是2-3连涨，这个数字越大说明下一天跌的风险越大
* 最近连续下跌天数数量 值域 0-N  通常是4-5连涨，这个数字越大说明下一天上涨的可能性越大
* 9日上涨数量比率 = 最近N日内红柱数量之和 / 最近N日内阴柱数量之和  值域 -1 to +1
* 9日上涨幅度比率 = 最近N日内红柱涨幅之和 / 最近N日内阴柱涨幅之和  值域 -1 to +1
* 21日上涨数量比率 = 最近N日内红柱数量之和 / 最近N日内阴柱数量之和 值域 -1 to +1
* 21日上涨幅度比率 = 最近N日内红柱涨幅之和 / 最近N日内阴柱涨幅之和 值域 -1 to +1
* 当天交易量和9日平均日交易量的(3/4的)位置关系
* 当天交易量和21日平均日交易量的(3/4的)位置关系，注意由于用的是3小时的数据，所以平均交易量取值应该是 3/4 才合理

### 关于微观环境描述的特征
```
其中最后一日的信息必须是残缺信息，
因为实盘的时候需要至少提前30分钟对明天收盘做出判断
关于最后一日的信息 开盘价是已知的，收盘价是最后一刻的价格，
最高价和最低价要动态计算一下
```
* 之前三日K线的涨幅 和 成交量
* 最近3日K线的形状 和 成交量 
* 关于成交量的记录应该理解为是能量，所以阳柱子就是正能量，阴柱是负能量
* 最后一日的三个小时涨幅变化
* MA5 和10日前的自身位置关系（斜率）
* 当前价格（最后收盘）相对MA5的位置关系
* 当前价格相对MA20，
* 当前价格MA60的位置关系
* 和上一个价格最低点的相对高度差
* KDJ的J数值

## Output result
先不离散化数据，用SVM尝试对其进行分类
* own < -1% 表示为：[1,0,0]  
* Flat +1% to -1% 表示为：[0,1,0]
* Up > +1% 表示为： [0,0,1]


## 提高准确率
用三种不同的预测机制共同投票，叠加每种可能性的概率，然后设置阈值，
最大概率应大于2/3 才做判断，否则放弃本次预测。三种结构不同的分类算法如下：
* 深度学习DNN
* SVM
* 随机森林

## 要过滤掉的事件
* 注意把包含跌幅大约10%的数据扔掉 可能包含了增发
* 把时间上不连续的股票扔掉 可能有停牌 如何判断呢？ 用股票交易日判断
* 历史不足60个交易日的扔掉
* 前20个交易日中有停盘或者增发的（大约30%异动的）扔掉

## 参考资料
* https://www.google.com/patents/US7043449

## 思考问题
* 应该用今天的聚焦点去比较 还是 收盘价？ 
```  
    看来半天没发现聚焦点的比较有什么规律 还是用收盘价吧
    压力位不是看最低价 而是看聚焦点之间的变化
    这样说只有最后一日的聚焦点才比较有分析价值
    算了 还是没看出来必然逻辑
```
* 应该什么时候重新训练模型呢？ 
```
    应该不是每天，应该是大熊市和大牛市的区分会改变模型的准确性，
    但是用什么信号来区分这个比例呢，
    最近一个月涨跌比例？ 
    最近一个月的大盘涨家/跌家比例
    如果5日均值超过30日均值 50% 那么重新训练模型
```

### 4D空间思想
如果我们以5日为一个单位，判断是否有突破其实应该理解为5日交易量的聚焦点
因此应当先算出聚焦点，然后在计算最后一个收盘价距离这个聚焦点的距离
负数算作是低于聚焦点，正数则是高于聚焦点

## 关于选股
可以尝试对alpha率进行动能弥散的计算来判断短期未来，也可以用此来对股票进行排序

[系统架构](docs/arch.md)