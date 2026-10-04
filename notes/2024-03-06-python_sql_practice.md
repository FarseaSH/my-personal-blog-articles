---
date: 2023-12-03T20:56:00+08:00
title: 用Python进行SQL练习的简单方法
category: 想法与笔记

tags: 
 - Python
 - SQL

toc: false
toc_fold: false

updated: 
copyright: true  # 暂不支持
---

在进行数据分析/数据科学方向上求职时，SQL题目的练习是不可或缺的。对于自己所写的 SQL 代码，最好的验证方式是跑出代码的数据结果。常见的刷题平台（如牛客网，leetcode）都支持这样的功能。但不在这些刷题平台的SQL题目，想要去运行自己所写的答案，就需要搭建一个能运行 SQL 的环境（如本地MySQL），这对于非技术背景的同学可能会比较困难。我最近找到一个简单的方案，只要能运行Python，安装相关包后，就能运行 SQL、进行 SQL 练习。本文该方案进行介绍。

<!--more-->

方案非常简单：使用Pandas载入数据，再利用`pandasql`包，即可使用SQL语句操作Pandas数据，执行SQL来得到返回数据。整个操作流程如下：

 首先，利用 pip 安装 `pandas` 和  `pandasql` Python 包

```bash
pip install pandas, pandasql
```

在 Python 环境中，将数据存储到 pandas `DataFrame`中

```python
df = pd.DataFrame([
    ('Adam', 18),
    ('Bod', 20),
    ('John', 35),
    ('David', 40),
    ('Frank', 47)
], columns=['name', 'age'])

# >>> df
#     name  age
# 0   Adam   18
# 1    Bod   20
# 2   John   35
# 3  David   40
# 4  Frank   47
```

用一个变量存下想要运行的SQL语句。需要注意的是，SQL语句中 From 后面跟随的表名为上面DataFrame对应的变量名（这里即为`df`）

```python
SQL = """
SELECT
    *
FROM
    df
WHERE
    age < 30
"""
```

用下面的语句运行SQL

```python
from pandasql import sqldf
pysqldf = lambda q: sqldf(q, globals())

pysqldf(sql)

# 运行结果为：
#    name  age
# 0  Adam   18
# 1   Bod   20
```

需要注意的是，pandasql背后使用的是SQLite引擎执行运算，SQLite在一些函数的语法上与HiveSQL略有不同，比如，HiveSQL中的`datediff`函数在sqlite中需要用`Cast(JulianDay(date1) - JulianDay(date2) AS Integer)`。如果想完全使用HiveSQL语法，可以搭建一个本地Spark环境，使用SparkSQL运行结果，后续我可能再写一篇文章单独介绍。

Leetcode 题目中的题目会给出形如下方这样展示表数据的样例文本

```
+----+-------+
| id | name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
```

为了练习方便，这里写了一个简单的 Python 函数，将这些样例文本转换为 pandas DataFrame

```python
def clean_leetcode_table(table_str: str):
    """
    将Leetcode中出现的表(如下格式)，解析转换为pandas DataFrame

    +----+-------+
    | id | score |
    +----+-------+
    | 1  | 3.50  |
    | 2  | 3.65  |
    | 3  | 4.00  |
    | 4  | 3.85  |
    | 5  | 4.00  |
    | 6  | 3.65  |
    +----+-------+
    """

    table_str = table_str.strip()

    header = None
    data_lst = []
    for line in table_str.split('\\n'):
        if "+" in line: continue

        content = [token.strip() for token in line.strip(' |').split('|')]

        if header is None:
            header = content
            continue

        data_lst.append(content)
    
    # 进行数值格式转换
    result = pd.DataFrame(data_lst, columns=header)
    for series_name in pd.DataFrame(data_lst, columns=header):
        result[series_name] = pd.to_numeric(result[series_name], errors='ignore')
    return result
```

## 完整代码

以练习[586. 订单最多的客户](https://leetcode.cn/problems/customer-placing-the-largest-number-of-orders/description/)为例

```python
import pandas as pd
from pandasql import sqldf

pysqldf = lambda q: sqldf(q, globals())

def clean_leetcode_table(table_str: str):
    """
    将Leetcode中出现的表(如下格式)，解析转换为pandas DataFrame

    +----+-------+
    | id | score |
    +----+-------+
    | 1  | 3.50  |
    | 2  | 3.65  |
    | 3  | 4.00  |
    | 4  | 3.85  |
    | 5  | 4.00  |
    | 6  | 3.65  |
    +----+-------+
    """

    table_str = table_str.strip()

    header = None
    data_lst = []
    for line in table_str.split('\\n'):
        if "+" in line: continue

        content = [token.strip() for token in line.strip(' |').split('|')]

        if header is None:
            header = content
            continue

        data_lst.append(content)
    
    # 进行数值格式转换
    result = pd.DataFrame(data_lst, columns=header)
    for series_name in pd.DataFrame(data_lst, columns=header):
        result[series_name] = pd.to_numeric(result[series_name], errors='ignore')
    return result

table_str = """
+--------------+-----------------+
| order_number | customer_number |
+--------------+-----------------+
| 1            | 1               |
| 2            | 2               |
| 3            | 3               |
| 4            | 3               |
+--------------+-----------------+
"""

Orders = clean_leetcode_table(table_str)

SQL = """
SELECT
    customer_number
FROM
    Orders
GROUP BY 1
ORDER BY COUNT(*) DESC
LIMIT 1
"""

pysqldf(sql)

# 返回结果：
# | customer_number |
# | --------------- |
# | 3               |


```





