# Python Pandas 测试工程师精通指南

> 标签: #python #pandas #data-analysis #testing #面试
> 创建时间: 2026-02-26
> 来源: [Pandas官方文档](https://pandas.pydata.org/) | [Pandas Tutorial](https://pandas.pydata.org/docs/user_guide/index.html)

## 概述

Pandas 是 Python 数据分析核心库，测试工程师常用它处理测试数据、数据比对、生成测试报告等场景。本文整理测试面试高频 Pandas 知识点，按重要程度分类。

---

## 一、知识体系总览

### 掌握程度分类

| 级别 | 说明 | 面试权重 |
|------|------|----------|
| 🔴 必须掌握 | 面试必问，日常必用 | 40% |
| 🟠 重要 | 常见考点，需要熟练 | 30% |
| 🟡 常用 | 工作中频繁使用 | 20% |
| 🟢 了解 | 高级场景，知道即可 | 10% |

---

## 二、核心知识点

### 🔴 必须掌握

#### 1. 数据结构

```python
import pandas as pd
import numpy as np

# ========== Series 一维数据 ==========
s = pd.Series([1, 2, 3, 4, 5])
s = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
s = pd.Series({'a': 1, 'b': 2, 'c': 3})

# 常用属性
s.values          # 值数组
s.index           # 索引
s.dtype           # 数据类型
s.shape           # 形状

# ========== DataFrame 二维表格 ==========
# 从字典创建
df = pd.DataFrame({
    'name': ['张三', '李四', '王五'],
    'age': [25, 30, 35],
    'city': ['北京', '上海', '广州']
})

# 从列表创建
df = pd.DataFrame([
    ['张三', 25, '北京'],
    ['李四', 30, '上海']
], columns=['name', 'age', 'city'])

# 从 NumPy 创建
df = pd.DataFrame(np.random.randn(3, 4), columns=['A', 'B', 'C', 'D'])

# 常用属性
df.shape          # (行数, 列数)
df.columns        # 列名
df.index          # 行索引
df.dtypes         # 各列数据类型
df.values         # NumPy 数组
df.info()         # 基本信息
df.describe()     # 统计描述
```

#### 2. 数据读取与写入

```python
# ========== 读取文件 ==========
# CSV
df = pd.read_csv('data.csv')
df = pd.read_csv('data.csv', encoding='utf-8')
df = pd.read_csv('data.csv', sep='\t')           # TSV
df = pd.read_csv('data.csv', header=None)        # 无表头
df = pd.read_csv('data.csv', usecols=['col1', 'col2'])  # 指定列
df = pd.read_csv('data.csv', nrows=1000)         # 只读前 N 行

# Excel
df = pd.read_excel('data.xlsx')
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')
df = pd.read_excel('data.xlsx', sheet_name=['Sheet1', 'Sheet2'])  # 多 sheet

# JSON
df = pd.read_json('data.json')
df = pd.read_json('data.json', orient='records')

# SQL
import sqlite3
conn = sqlite3.connect('database.db')
df = pd.read_sql('SELECT * FROM users', conn)
df = pd.read_sql_table('users', conn)

# ========== 写入文件 ==========
df.to_csv('output.csv', index=False)
df.to_csv('output.csv', encoding='utf-8-sig')    # Excel 友好
df.to_excel('output.xlsx', index=False)
df.to_excel('output.xlsx', sheet_name='Sheet1')
df.to_json('output.json', orient='records', force_ascii=False)

# 多 sheet
with pd.ExcelWriter('output.xlsx') as writer:
    df1.to_excel(writer, sheet_name='Sheet1')
    df2.to_excel(writer, sheet_name='Sheet2')
```

#### 3. 数据选择与过滤

```python
# ========== 选择列 ==========
df['name']                    # 单列（Series）
df[['name', 'age']]           # 多列（DataFrame）
df.name                       # 属性方式（不推荐）

# ========== 选择行 ==========
# 位置索引
df.iloc[0]                    # 第 1 行
df.iloc[0:5]                  # 前 5 行
df.iloc[[0, 2, 4]]            # 指定行

# 标签索引
df.loc[0]                     # 索引为 0 的行
df.loc[0:5]                   # 索引 0-5 的行
df.loc[0, 'name']             # 指定单元格

# 混合
df.iloc[0:5, 0:2]             # 前 5 行，前 2 列
df.loc[0:5, ['name', 'age']]

# ========== 条件过滤 ==========
# 单条件
df[df['age'] > 25]
df[df['city'] == '北京']

# 多条件
df[(df['age'] > 25) & (df['city'] == '北京')]
df[(df['age'] > 25) | (df['city'] == '上海')]

# isin
df[df['city'].isin(['北京', '上海'])]

# 包含字符串
df[df['name'].str.contains('张')]

# 非空
df[df['email'].notna()]
df[df['email'].isna()]

# query 方法
df.query('age > 25 and city == "北京"')
df.query('city in ["北京", "上海"]')
```

#### 4. 数据清洗

```python
# ========== 缺失值处理 ==========
# 检查缺失值
df.isna().sum()               # 每列缺失数
df.isna().any()               # 哪些列有缺失
df.dropna()                   # 删除含缺失值的行
df.dropna(subset=['age'])     # 只检查指定列
df.dropna(how='all')          # 全为空才删除
df.dropna(axis=1)             # 删除含缺失值的列

# 填充缺失值
df.fillna(0)                          # 填充固定值
df.fillna({'age': 0, 'city': '未知'})  # 各列不同值
df.fillna(df.mean())                  # 填充均值
df.fillna(method='ffill')             # 前向填充
df.fillna(method='bfill')             # 后向填充

# ========== 重复值处理 ==========
df.duplicated()               # 检查重复（返回布尔）
df.duplicated().sum()         # 重复数量
df.drop_duplicates()          # 删除重复行
df.drop_duplicates(subset=['name'])  # 按列去重
df.drop_duplicates(keep='last')      # 保留最后一个

# ========== 数据类型转换 ==========
df['age'] = df['age'].astype(int)
df['date'] = pd.to_datetime(df['date'])
df['price'] = pd.to_numeric(df['price'], errors='coerce')  # 错误转 NaN

# ========== 字符串处理 ==========
df['name'].str.lower()        # 小写
df['name'].str.upper()        # 大写
df['name'].str.strip()        # 去空格
df['name'].str.replace('张', '王')
df['name'].str.split(',')     # 分割
df['name'].str.len()          # 长度
df['name'].str.startswith('张')
df['name'].str.contains('张')
```

#### 5. 数据统计分析

```python
# ========== 基础统计 ==========
df.describe()                  # 描述性统计
df.mean()                      # 均值
df.median()                    # 中位数
df.std()                       # 标准差
df.var()                       # 方差
df.sum()                       # 求和
df.count()                     # 计数
df.max(), df.min()             # 最大最小
df.quantile(0.25)              # 分位数

# ========== 单列统计 ==========
df['age'].mean()
df['age'].value_counts()       # 值计数
df['city'].value_counts()      # 分类计数
df['city'].unique()            # 唯一值
df['city'].nunique()           # 唯一值数量

# ========== 相关性 ==========
df.corr()                      # 相关系数矩阵
df.cov()                       # 协方差矩阵
```

---

### 🟠 重要

#### 6. 分组聚合 (GroupBy)

```python
# ========== 基础分组 ==========
# 单列分组
df.groupby('city')['age'].mean()
df.groupby('city')['age'].agg(['mean', 'max', 'min'])

# 多列分组
df.groupby(['city', 'gender'])['age'].mean()

# 多列聚合
df.groupby('city').agg({
    'age': ['mean', 'max'],
    'salary': ['sum', 'mean']
})

# ========== 自定义聚合 ==========
df.groupby('city').agg({
    'age': lambda x: x.max() - x.min()
})

# ========== 分组后操作 ==========
grouped = df.groupby('city')
grouped.get_group('北京')      # 获取某组
grouped.size()                 # 各组大小
grouped.count()                # 各组非空计数

# ========== transform ==========
# 返回与原 DataFrame 相同形状
df['avg_age'] = df.groupby('city')['age'].transform('mean')

# ========== apply ==========
# 自定义函数
df.groupby('city').apply(lambda x: x.nlargest(3, 'age'))
```

#### 7. 数据合并

```python
# ========== concat 拼接 ==========
# 上下拼接
pd.concat([df1, df2])
pd.concat([df1, df2], ignore_index=True)

# 左右拼接
pd.concat([df1, df2], axis=1)

# ========== merge 连接 ==========
# 内连接（默认）
pd.merge(df1, df2, on='user_id')
pd.merge(df1, df2, on=['user_id', 'order_id'])

# 左连接
pd.merge(df1, df2, on='user_id', how='left')

# 其他连接
pd.merge(df1, df2, on='user_id', how='right')  # 右连接
pd.merge(df1, df2, on='user_id', how='outer')  # 外连接

# 不同列名
pd.merge(df1, df2, left_on='id', right_on='user_id')

# ========== join ==========
df1.join(df2, on='key', lsuffix='_left', rsuffix='_right')

# ========== append（已弃用，用 concat）==========
# pd.concat([df1, df2], ignore_index=True)
```

#### 8. 数据透视表

```python
# ========== pivot_table ==========
# 基础透视表
pd.pivot_table(df, values='sales', index='city', columns='year', aggfunc='sum')

# 多值多聚合
pd.pivot_table(df,
    values=['sales', 'profit'],
    index='city',
    columns='year',
    aggfunc={'sales': 'sum', 'profit': 'mean'}
)

# 添加小计
pd.pivot_table(df, values='sales', index='city', columns='year',
               aggfunc='sum', margins=True)

# ========== crosstab 交叉表 ==========
pd.crosstab(df['city'], df['gender'])
pd.crosstab(df['city'], df['gender'], margins=True)
pd.crosstab(df['city'], df['gender'], normalize='index')  # 按行归一化
```

#### 9. 时间序列处理

```python
# ========== 日期转换 ==========
df['date'] = pd.to_datetime(df['date'])
df['date'] = pd.to_datetime(df['date'], format='%Y-%m-%d')

# ========== 提取日期属性 ==========
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day'] = df['date'].dt.day
df['weekday'] = df['date'].dt.weekday          # 0=周一
df['hour'] = df['date'].dt.hour
df['quarter'] = df['date'].dt.quarter
df['is_weekend'] = df['date'].dt.weekday >= 5

# ========== 日期范围 ==========
pd.date_range('2024-01-01', '2024-12-31', freq='D')    # 每天
pd.date_range('2024-01-01', periods=12, freq='M')      # 12 个月
pd.date_range('2024-01-01', periods=10, freq='W')      # 10 周

# ========== 重采样 ==========
# 时间序列聚合
df.resample('D', on='date').sum()        # 按天
df.resample('W', on='date').mean()       # 按周
df.resample('M', on='date').sum()        # 按月

# ========== 时间窗口 ==========
df.rolling(window=7).mean()              # 7 天移动平均
df.expanding().sum()                     # 累计求和
```

---

### 🟡 常用

#### 10. 数据排序与排名

```python
# ========== 排序 ==========
df.sort_values('age')                    # 按值排序
df.sort_values('age', ascending=False)   # 降序
df.sort_values(['city', 'age'])          # 多列排序
df.sort_index()                          # 按索引排序

# ========== 排名 ==========
df['rank'] = df['score'].rank()          # 排名
df['rank'] = df['score'].rank(ascending=False)
df['rank'] = df['score'].rank(method='dense')  # 紧凑排名

# nlargest / nsmallest
df.nlargest(10, 'score')                 # 前 10 大
df.nsmallest(10, 'score')                # 前 10 小
```

#### 11. 数据重塑

```python
# ========== melt 宽变长 ==========
df_melted = pd.melt(df, id_vars=['name'], value_vars=['math', 'english'])
df_melted = df.melt(id_vars=['name'], var_name='subject', value_name='score')

# ========== pivot 长变宽 ==========
df_pivot = df.pivot(index='name', columns='subject', values='score')

# ========== stack / unstack ==========
df.stack()                               # 列转行
df.unstack()                             # 行转列

# ========== transpose ==========
df.T                                     # 转置
```

#### 12. 数据修改

```python
# ========== 赋值 ==========
df['new_col'] = 0                        # 新列
df['new_col'] = df['col1'] + df['col2']
df.loc[0, 'name'] = '新名字'             # 修改单元格

# ========== apply ==========
df['name_upper'] = df['name'].apply(str.upper)
df['category'] = df['age'].apply(lambda x: '青年' if x < 30 else '中年')

# ========== map ==========
df['gender_cn'] = df['gender'].map({'M': '男', 'F': '女'})

# ========== replace ==========
df['status'].replace({'active': 1, 'inactive': 0})

# ========== assign ==========
df = df.assign(
    total=df['col1'] + df['col2'],
    category='A'
)
```

---

### 🟢 了解

#### 13. 高级特性

```python
# ========== 多层索引 ==========
df = pd.DataFrame(np.random.randn(4, 2),
    index=[['A', 'A', 'B', 'B'], [1, 2, 1, 2]],
    columns=['X', 'Y'])
df.loc['A', 1]

# ========== 样式 ==========
df.style.highlight_max()
df.style.background_gradient()

# ========== 与 SQL 交互 ==========
import pandasql as ps
ps.sqldf("SELECT * FROM df WHERE age > 25")

# ========== 性能优化 ==========
# 使用 dtype 减少内存
df = pd.read_csv('data.csv', dtype={'id': 'int32', 'name': 'category'})

# 分块读取
for chunk in pd.read_csv('big.csv', chunksize=10000):
    process(chunk)
```

---

## 三、面试高频问题

### 基础篇

| 问题 | 答案 |
|------|------|
| Series 和 DataFrame 区别？ | Series 一维，DataFrame 二维表格 |
| 如何读取 CSV？ | `pd.read_csv('file.csv')` |
| 如何处理缺失值？ | `dropna()` 删除，`fillna()` 填充 |
| 如何去重？ | `drop_duplicates()` |
| iloc 和 loc 区别？ | iloc 位置索引，loc 标签索引 |

### 进阶篇

| 问题 | 答案 |
|------|------|
| groupby 的工作流程？ | 分割 → 应用 → 合并 |
| merge 和 join 区别？ | merge 更灵活，join 基于索引 |
| 如何实现 SQL 的 GROUP BY？ | `df.groupby().agg()` |
| 如何处理大数据？ | chunksize 分块、dtype 优化 |
| map、apply、applymap 区别？ | map 元素级，apply 行/列级，applymap 全元素 |

### 高级篇

| 问题 | 答案 |
|------|------|
| 如何优化 Pandas 性能？ | dtype、向量化、避免循环 |
| 如何处理时间序列？ | `to_datetime()`、`resample()` |
| pivot_table 和 pivot 区别？ | pivot_table 支持聚合 |

---

## 四、实战场景

### 场景1：测试数据比对

```python
# 比对两个 Excel 文件
df1 = pd.read_excel('expected.xlsx')
df2 = pd.read_excel('actual.xlsx')

# 方式1：直接比较
diff = df1.compare(df2)

# 方式2：合并后找差异
merged = pd.merge(df1, df2, how='outer', indicator=True)
diff = merged[merged['_merge'] != 'both']

# 方式3：逐列比较
for col in df1.columns:
    if not df1[col].equals(df2[col]):
        print(f"列 {col} 不一致")
```

### 场景2：生成测试数据

```python
# 生成随机测试数据
import random

n = 1000
df = pd.DataFrame({
    'user_id': range(1, n + 1),
    'name': [f'用户{i}' for i in range(1, n + 1)],
    'age': np.random.randint(18, 60, n),
    'city': np.random.choice(['北京', '上海', '广州', '深圳'], n),
    'amount': np.round(np.random.uniform(100, 10000, n), 2),
    'created_at': pd.date_range('2024-01-01', periods=n, freq='H')
})

df.to_csv('test_data.csv', index=False)
```

### 场景3：数据验证

```python
# 验证数据质量
def validate_data(df):
    issues = []

    # 检查空值
    null_counts = df.isna().sum()
    for col, count in null_counts.items():
        if count > 0:
            issues.append(f"列 {col} 有 {count} 个空值")

    # 检查重复
    dup_count = df.duplicated().sum()
    if dup_count > 0:
        issues.append(f"发现 {dup_count} 条重复数据")

    # 检查异常值
    if (df['age'] < 0).any():
        issues.append("年龄存在负值")

    return issues
```

### 场景4：数据统计报告

```python
# 生成统计报告
def generate_report(df):
    report = {
        '总行数': len(df),
        '总列数': len(df.columns),
        '缺失值统计': df.isna().sum().to_dict(),
        '数值列统计': df.describe().to_dict(),
        '唯一值统计': {col: df[col].nunique() for col in df.columns}
    }
    return report
```

---

## 五、Pandas 速查表

### 数据结构

| 操作 | 代码 |
|------|------|
| 创建 DataFrame | `pd.DataFrame(dict)` |
| 查看 | `df.head()`, `df.tail()`, `df.info()` |
| 形状 | `df.shape` |
| 列名 | `df.columns` |

### 选择

| 操作 | 代码 |
|------|------|
| 选列 | `df['col']`, `df[['col1', 'col2']]` |
| 选行 | `df.iloc[i]`, `df.loc[i]` |
| 条件 | `df[df['col'] > value]` |

### 清洗

| 操作 | 代码 |
|------|------|
| 删除空值 | `df.dropna()` |
| 填充空值 | `df.fillna(value)` |
| 去重 | `df.drop_duplicates()` |
| 类型转换 | `df['col'].astype(type)` |

### 统计

| 操作 | 代码 |
|------|------|
| 描述统计 | `df.describe()` |
| 均值 | `df['col'].mean()` |
| 计数 | `df['col'].value_counts()` |
| 分组 | `df.groupby('col').agg()` |

### 合并

| 操作 | 代码 |
|------|------|
| 拼接 | `pd.concat([df1, df2])` |
| 连接 | `pd.merge(df1, df2, on='key')` |
| 透视 | `pd.pivot_table()` |

---

## 相关知识点

- [[Python Requests 精通指南]]
- [[SQL 命令测试工程师精通指南]]
- [[Pytest 面试完全指南]]

---
*采集自 Claude Code 对话*

**Sources:**
- [Pandas 官方文档](https://pandas.pydata.org/)
- [30道Pandas高频考点全解析](https://blog.csdn.net/Rikki1013/article/details/150548297)
- [Python数据分析面试](https://www.fanruan.com/blog/article/1775901/)
