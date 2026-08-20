# Python 数据分析入门 / Python Data Analysis Basics

## 一、为什么学 Python 数据分析 / Why Learn Python for Data Analysis

Python 是数据科学领域最常用的编程语言，留学数据科学专业必备。

Python is the most popular language in data science. Essential for Data Science master's programs.

| 库 Library | 用途 Purpose |
|------|------|
| NumPy | 数值计算 / Numerical computing |
| Pandas | 数据处理 / Data processing |
| Matplotlib | 数据可视化 / Data visualization |
| Scikit-learn | 机器学习 / Machine learning |

## 二、环境准备 / Environment Setup

### 安装 / Installation

```bash
# 推荐 Anaconda（包含所有数据科学库）
# Recommended: Anaconda (includes all data science libraries)

# 或用 pip 安装 / Or install with pip
pip install numpy pandas matplotlib scikit-learn jupyter
```

### Jupyter Notebook

```bash
# 启动 / Start
jupyter notebook

# 或 Jupyter Lab（更现代的界面）/ Or Jupyter Lab (modern UI)
jupyter lab
```

Jupyter Notebook 是交互式编程环境，适合数据探索和可视化。

Jupyter Notebook is an interactive environment, ideal for data exploration and visualization.

## 三、NumPy 基础 / NumPy Basics

### 创建数组 / Create Arrays

```python
import numpy as np

# 一维数组 / 1D array
arr1 = np.array([1, 2, 3, 4, 5])

# 二维数组 / 2D array
arr2 = np.array([[1, 2, 3],
                 [4, 5, 6]])

# 特殊数组 / Special arrays
zeros = np.zeros(5)         # [0, 0, 0, 0, 0]
ones = np.ones((2, 3))      # 2x3 全1矩阵 / 2x3 all ones
range_arr = np.arange(0, 10, 2)  # [0, 2, 4, 6, 8]
```

### 数组运算 / Array Operations

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# 元素级运算 / Element-wise operations
a + b          # [5, 7, 9]
a * b          # [4, 10, 18]
a ** 2         # [1, 4, 9]

# 统计 / Statistics
a.mean()       # 平均值 / Mean: 2.0
a.sum()        # 求和 / Sum: 6
a.max()        # 最大值 / Max: 3
a.min()        # 最小值 / Min: 1
a.std()        # 标准差 / Standard deviation
```

### 索引和切片 / Indexing & Slicing

```python
arr = np.array([10, 20, 30, 40, 50])

arr[0]         # 10（第一个 / First）
arr[-1]        # 50（最后一个 / Last）
arr[1:4]       # [20, 30, 40]（切片 / Slice）

# 二维 / 2D
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
matrix[0, 0]   # 1（第0行第0列 / Row 0, Col 0）
matrix[1, :]   # [4, 5, 6]（第1行所有列 / Row 1, all columns）
matrix[:, 1]   # [2, 5, 8]（所有行第1列 / All rows, Col 1）
```

## 四、Pandas 基础 / Pandas Basics

### Series（一维）/ Series (1D)

```python
import pandas as pd

s = pd.Series([10, 20, 30, 40], index=['a', 'b', 'c', 'd'])

s['a']         # 10
s.sum()        # 100
s.mean()       # 25.0
s[s > 15]      # 筛选大于15的 / Filter > 15: b=20, c=30, d=40
```

### DataFrame（二维表）/ DataFrame (2D Table)

#### 创建 / Create

```python
# 从字典创建 / Create from dict
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'city': ['Beijing', 'Shanghai', 'Guangzhou']
})

# 从 CSV 读取 / Read from CSV
df = pd.read_csv('data.csv')

# 从 Excel 读取 / Read from Excel
df = pd.read_excel('data.xlsx')
```

#### 查看数据 / View Data

```python
df.head()          # 前5行 / First 5 rows
df.head(10)        # 前10行 / First 10 rows
df.tail()          # 后5行 / Last 5 rows
df.shape           # (行数, 列数) / (rows, cols)
df.columns         # 列名 / Column names
df.dtypes          # 数据类型 / Data types
df.info()          # 概要信息 / Summary info
df.describe()      # 统计摘要 / Statistical summary
```

#### 选择数据 / Select Data

```python
# 选列 / Select column
df['name']              # 选一列 / One column
df[['name', 'age']]     # 选多列 / Multiple columns

# 选行 / Select rows
df.iloc[0]              # 第0行（按位置）/ Row 0 (by position)
df.iloc[0:3]            # 前3行 / First 3 rows
df.loc[0]               # 第0行（按标签）/ Row 0 (by label)

# 条件筛选 / Conditional filter
df[df['age'] > 25]                     # 年龄大于25 / Age > 25
df[(df['age'] > 25) & (df['city'] == 'Beijing')]  # 多条件 / Multiple conditions
```

#### 数据处理 / Data Processing

```python
# 缺失值 / Missing values
df.isnull()                  # 检查缺失 / Check missing
df.isnull().sum()            # 每列缺失数 / Missing count per column
df.dropna()                  # 删除缺失行 / Drop missing rows
df.fillna(0)                 # 填充0 / Fill with 0
df['age'].fillna(df['age'].mean())  # 用平均值填充 / Fill with mean

# 排序 / Sort
df.sort_values('age')                    # 按年龄升序 / Sort by age ascending
df.sort_values('age', ascending=False)   # 降序 / Descending

# 分组聚合 / Group by & aggregate
df.groupby('city')['age'].mean()         # 按城市分组求平均年龄 / Avg age by city
df.groupby('city').agg({
    'age': ['mean', 'max', 'min'],
    'name': 'count'
})

# 新增列 / Add column
df['age_group'] = df['age'].apply(lambda x: 'young' if x < 30 else 'old')
```

## 五、Matplotlib 可视化 / Matplotlib Visualization

### 基本图表 / Basic Charts

```python
import matplotlib.pyplot as plt

# 折线图 / Line chart
plt.plot([1, 2, 3, 4], [10, 20, 25, 30])
plt.title('Sales Trend / 销售趋势')
plt.xlabel('Month / 月份')
plt.ylabel('Sales / 销量')
plt.show()

# 柱状图 / Bar chart
cities = ['Beijing', 'Shanghai', 'Guangzhou']
values = [100, 150, 80]
plt.bar(cities, values)
plt.title('Sales by City / 各城市销量')
plt.show()

# 散点图 / Scatter plot
plt.scatter(df['age'], df['salary'])
plt.xlabel('Age / 年龄')
plt.ylabel('Salary / 薪资')
plt.show()

# 直方图 / Histogram
plt.hist(df['age'], bins=10)
plt.title('Age Distribution / 年龄分布')
plt.show()
```

### Pandas 内置绘图 / Pandas Built-in Plot

```python
# 更简洁的画法 / Simpler approach
df['age'].hist(bins=10)              # 直方图 / Histogram
df.plot(x='age', y='salary', kind='scatter')  # 散点图 / Scatter
df.groupby('city')['age'].mean().plot(kind='bar')  # 柱状图 / Bar
```

## 六、实战示例 / Practical Example

```python
import pandas as pd
import matplotlib.pyplot as plt

# 1. 读取数据 / Load data
df = pd.read_csv('sales_data.csv')

# 2. 查看数据 / Explore data
print(df.head())
print(df.info())
print(df.describe())

# 3. 数据清洗 / Clean data
df = df.dropna()                    # 删除缺失值 / Drop missing
df = df[df['price'] > 0]           # 过滤异常值 / Filter outliers

# 4. 分析 / Analyze
monthly_sales = df.groupby('month')['revenue'].sum()
print(monthly_sales)

# 5. 可视化 / Visualize
monthly_sales.plot(kind='line', title='Monthly Revenue / 月收入')
plt.xlabel('Month / 月份')
plt.ylabel('Revenue / 收入')
plt.show()

# 6. 保存结果 / Save results
monthly_sales.to_csv('monthly_summary.csv')
```

## 七、学习路线 / Learning Path

```
1. Python 基础语法 / Python basics (1-2 weeks)
   → 变量、循环、函数、列表、字典
   → Variables, loops, functions, lists, dicts

2. NumPy / NumPy (1 week)
   → 数组操作、运算
   → Array operations, math

3. Pandas / Pandas (2 weeks)
   → DataFrame 操作、数据清洗
   → DataFrame operations, data cleaning

4. Matplotlib / Matplotlib (1 week)
   → 基本图表
   → Basic charts

5. 统计学基础 / Statistics basics (2 weeks)
   → 均值、方差、分布、假设检验
   → Mean, variance, distributions, hypothesis testing

6. Scikit-learn / Scikit-learn (2-3 weeks)
   → 回归、分类、聚类
   → Regression, classification, clustering
```

## 八、常用速查 / Quick Reference

| 操作 Operation | 代码 Code |
|------|------|
| 读 CSV Read CSV | `pd.read_csv('file.csv')` |
| 写 CSV Write CSV | `df.to_csv('output.csv', index=False)` |
| 查看形状 Shape | `df.shape` |
| 查看类型 Types | `df.dtypes` |
| 统计摘要 Summary | `df.describe()` |
| 分组 Group | `df.groupby('col')` |
| 排序 Sort | `df.sort_values('col')` |
| 去重 Unique | `df['col'].unique()` |
| 计数 Count | `df['col'].value_counts()` |
| 应用函数 Apply | `df['col'].apply(func)` |
