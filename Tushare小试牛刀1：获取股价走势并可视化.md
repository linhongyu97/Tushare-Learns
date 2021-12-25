# Tushare小试牛刀1：获取股价走势并可视化

个人Tushare ID：414283



## Tushare简介

Tushare是一个为各类金融投资和研究人员提供数据和工具的平台，其数据内容涵盖了股票、基金、期货、债券、外汇、行业大数据等，能够满足有一定编程基础人员的基本分析需求。

笔者正是通过申请学生身份后得到了免费的token，基于此来做一些基本的股票分析，下面将介绍如何使用python语言，通过Tushare接口来获取个股的走势，并将走势可视化为折线图。



## 使用Tushare获取股价走势并可视化

具体的程序逻辑可以概括为：

1. 导入相应的包并设置好token
2. 通过数据接口`pro.daily()`获取个股的日线行情数据，并存为dataframe结构。
3. 通过dataframe内置的plot可视化为折线图。



下面直接展示代码（已封装为函数）：

```python
import pandas as pd

import matplotlib.pyplot as plt
%matplotlib inline
# 正常显示画图时出现的中文和负号
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

import tushare as ts
# 使用之前先输入token，可以从个人主页上复制出来，
token = '***'
ts.set_token(token)
pro = ts.pro_api()

# 根据股票代码 获取 该股票日线收盘价格 的 折线图走势
def plot_stock_daily_close_price(str_code):
    df = pro.daily(ts_code=str_code)
    plt.figure(figsize=(8,6))
    df.plot(x='trade_date', y='close')
    plt.title(str_code+"的日线收盘股价走势")
    plt.ylabel("股价")
    plt.xlabel("日期")

# 传入参数为 股票代码
plot_stock_daily_close_price('000001.SZ')
```

效果图如下：

![image-20211225213748603](https://s2.loli.net/2021/12/25/oBS2e6ZpPHmwiFn.png)



感谢您的阅读！如有指导请提issue~
