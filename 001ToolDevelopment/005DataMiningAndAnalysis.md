## 数据分析与挖掘

### 数据分析概述
    定义：
    简单来说，数据分析就是对数据进行分析。专业的说法，数据分析是指根据分析目的，用适当的
    统计分析方法及工具，对收集来的数据进行处理与分析，提取有价值的信息发挥数据的作用；
    
    作用：
    主要实现三大作用：现状分析、原因分析、预测分析（定量）。数据分析目标明确，先做假设，
    然后通过数据分析来验证假设是否正确，从而得到相应的结论
    
### 数据挖掘概述
    定义：
    数据挖掘就是从大量数据中，通过统计学、人工智能、机器学习等方法，挖掘出未知的，具有
    价值的信息和知识的过程；
    
    作用：
    数据挖掘主要侧重解决四类问题：分类、聚类、关联和预测（定量定性），数据挖掘的重点在
    寻找未知的模式与规律；事先未知的，但又非常有价值的信息
    
### 数据挖掘场景
    分类：对客户等级进行划分、验证码识别、水果品质自动筛选等
    回归：对连续型数据进行预测、趋势预测等
    聚类：客户价值预测、商圈预测等
    关联分析：超市货品摆放、个性化推荐等
    
### 数据挖掘常用库
    numpy模块：用于矩阵运算、随机数的生成等
    pandas模块：用于数据的读取、清洗、整理、运算、可视化等
    matplotlib模块：专用于数据可视化，当前含有统计类的seaborn模块
    statsmodels模块：用于构建模型、如线性回归、岭回归、逻辑回归、主成分分析等
    scipy模块：专用于统计中的各种假设校验、如卡方校验、相关系数校验、正态性校验、t校验、F校验等
    sklearn模块：专用于机器学习、包含了常规的数据挖掘算法，如决策树、森林树、提升树、贝叶斯、K近邻、SVM、GBDT、Kmeans等
    
### 需求分析
    版本：V1.0
    需求：预测模型，预测股票的波动趋势
    功能描述：
        雪球财经网股票数据爬取
        数据指标分析
        数据可视化显示
        数据趋势预估
        
```python
# coding=utf-8

"""
    数据分析操作流程
    1. 获取数据
    2. 存储数据
    3. 洗数据(过滤数据）
    4. 算法介入
    5. 结果展示
    6. 分析汇总
"""

import requests, time, csv, threading, pandas
import matplotlib.pyplot as plt
from pylab import mpl


# 创建数据存储文件csv，newline=''表示换行空格
fo = open('股票数据.csv', mode='w', encoding='utf-8', newline='')
# 写入csv列名
csv_write = csv.DictWriter(fo, fieldnames=['股票名称', '股票代码', '当前价格', '成交量'])
# 写入
csv_write.writeheader()


# 获取数据：1、本地获取；2、网站获取
# 格式化爬取多页数据, round(time.time()*1000)，乘以1000是因为要跟原来参数保留的位数一致
url_list = ['https://xueqiu.com/service/v5/stock/screener/quote/list?page={}&size=30&order=desc&orderby=percent&order_by=percent&market=CN&type=sh_sz&_=1612157509428'.format(page, round(time.time()*1000)) for page in range(1, 2)]

def request_data(url):
    # 一定添加请求头，否则会识别到python在爬取，因为网站存在防扒操作
    header = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:84.0) Gecko/20100101 Firefox/84.0'
    }
    resp = requests.get(url, headers=header)
    # print(resp.json())

    # 取出页面中需要的数据（如果是爬取内部大量数据可以使用sql语句）
    data_list = resp.json()['data']['list']
    for one in data_list:
        dict_tmp = {}
        dict_tmp['股票名称'] = one['name']
        dict_tmp['股票代码'] = one['symbol']
        dict_tmp['当前价格'] = one['current']
        dict_tmp['成交量'] = one['volume']
        print(dict_tmp)
        # 写入csv或者excel或者（存在大量数据可以写入数据库）—————— csv比较通用
        csv_write.writerow(dict_tmp)

def thread_request():
    threads = []
    for url in url_list:
        threads.append(threading.Thread(target=request_data, args=(url, )))

    for one in threads:
        one.start()
        time.sleep(1)

    for one in threads:
        one.join()

start_time = time.time()
thread_request()
# 需要关闭打开的线程，否则下面的pandas无法数据
fo.close()
end_time = time.time()
print('总共耗时>>>', end_time - start_time)

# 数据处理：不同场景需要不同的数据——再过滤数据
# 读取csv文件
data_df = pandas.read_csv('股票数据.csv', engine='python')
# 删除包含缺失值的行
df = data_df.dropna()
# 取出需要的某几列数据
df1 = df[['股票名称', '当前价格']]
# 切片取一部分的数据
df2 = df1.iloc[:10]
print(df2)

# 算法分析 ：引入算法--行业内部--经验的操作

# 设置图形中文正确显示
mpl.rcParams['font.sans-serif'] = ['SimHei']
mpl.rcParams['axes.unicode_minus'] = False

# 绘制图形
plt.bar(df2['股票名称'].values, height=df2['当前价格'].values, label='股票当前价')
# 设置x轴标签旋转角度
plt.xticks(rotation=-18)
# 使用zip函数把股票名称和股票价格对应起来
for a, b in zip(df2['股票名称'].values, df2['当前价格'].values):
    print(a, b)
    plt.text(a, b+1, b, horizontalalignment='center', verticalalignment='bottom')



# 设置生效
plt.legend()
# 设置x轴标签值
plt.xlabel('股票名称')
# 设置y轴标签值
plt.ylabel('当前价格')
# 展示
plt.show()


if __name__ == '__main__':
    request_data(url_list[0])
```    