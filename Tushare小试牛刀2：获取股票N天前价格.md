# Tushare小试牛刀2：获取股票N天前的价格

获取股票N天前的收盘价，与当前价位相比，是一些考虑动量的选股策略的基础，比如通过比较当前交易日的收盘价与20天前的收盘价，计算涨幅再进行排序，可以找到N天内“市场最靓的仔”。

下面使用Python代码进行实现：

```python
import pandas as pd
pd.set_option('display.max_rows', 1000)
import numpy as np
import tushare as ts

from datetime import datetime
from dateutil.relativedelta import relativedelta

import matplotlib.pyplot as plt
%matplotlib inline
# 正常显示画图时出现的中文和负号
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

import tushare as ts
# 使用之前先输入token，可以从个人主页上复制出来，
# 每次调用数据需要先运行该命令
token = '****'
ts.set_token(token)
pro = ts.pro_api()

# 获取 股票ID 及 基本信息
df_all_stock_basic_info = pro.stock_basic(exchange='', list_status='L', fields='ts_code,symbol,name,list_status,area,industry,exchange,list_date')

# 获取 股票 的 一级行业
df_all_stock_industry = pd.read_excel('./股票RPS强度实现/最新个股申万行业分类(完整版-截至7月末).xlsx').rename(columns={'股票代码':'ts_code'})
df_all_stock_industry = df_all_stock_industry[df_all_stock_industry['交易所'] == 'A股']
#df_all_stock_industry = df_all_stock_industry[['ts_code', '新版一级行业', '新版二级行业', '新版三级行业']]
df_all_stock_industry = df_all_stock_industry[['ts_code', '新版一级行业']]

df_all_stock_industry.head(2)


str_today = "".join(str(datetime.today())[0:10].split('-'))
df_trade_calender = pro.trade_cal(exchange='', start_date='20200101', end_date=str_today)
df_trade_calender_open = df_trade_calender[df_trade_calender['is_open'] == 1]
arr_trade_calender_open = df_trade_calender_open.cal_date.values[-20:]

def stock_incr(str_input_day):
    # 获取日期，用于圈定时间范围
    str_today = datetime.today().strftime(str_input_day)
    str_today_last_year = (datetime.today() + relativedelta(years=-1)).strftime('%Y%m%d')
    #print("最近一个交易日:", str_today, "去年同期的交易日：", str_today_last_year)

    # 获取近两年交易日期
    df_trade_calender = pro.trade_cal(exchange='', start_date=str_today_last_year, end_date=str_today)
    df_trade_calender_open = df_trade_calender[df_trade_calender['is_open'] == 1]
    arr_trade_calender_open = df_trade_calender_open.cal_date.values  # 获取近一年交易日期，存为数组

    # 获取上市时间大于1年的所有股票 用于统计 RPS
    df_all_stock_bigger_1_year_basic_info = df_all_stock_basic_info[(df_all_stock_basic_info['list_date'] < arr_trade_calender_open[0]) & (df_all_stock_basic_info['exchange'] != 'BSE')]
    df_rps_stock = df_all_stock_bigger_1_year_basic_info[['ts_code', 'name']]
    df_rps_stock = df_rps_stock.join(df_all_stock_industry.set_index('ts_code'), on='ts_code', how='left')

    # 拼接得到 20/50/120/250 日收盘价
    df_tmp = pro.daily(trade_date=arr_trade_calender_open[-1])[['ts_code', 'close']].rename(columns={'close': 'close_today'})
    df_rps_stock = df_rps_stock.join(df_tmp.set_index('ts_code'), on='ts_code', how='left')
    
    
    list_cal_date = [20, 50, 120, 250]
    for date in list_cal_date:
        if date != 250:
            df_tmp = pro.daily(trade_date=arr_trade_calender_open[-(date+1)])[['ts_code', 'close']].rename(columns={'close': 'close_'+str(date)})
        else:
            df_tmp = pro.daily(trade_date=arr_trade_calender_open[0])[['ts_code', 'close']].rename(columns={'close': 'close_'+str(date)})

        df_rps_stock = df_rps_stock.join(df_tmp.set_index('ts_code'), on='ts_code', how='left')
    
    return df_rps_stock
    
stock_incr('2022-03-01')
```


