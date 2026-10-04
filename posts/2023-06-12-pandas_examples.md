---
title: 在100个实践例子中学习Pandas
date: 2023-06-12T20:12:00+08:00
updated:
copyright: true  # 暂不支持
category: 经验
tags:
    - Pandas 
toc: true
toc_fold: false
draft: false
---

熟练掌握Pandas是一个比较漫长的过程，阅读入门教程或API文档只是上手的一部分。在实际操作中，数据各式各样，需要转换的格式也各不相同，在不熟练的情况下，一时间可能想不到用什么恰当的Pandas函数，导致写出的Pandas代码会比较复杂，既浪费时间也让代码难以阅读。一个比较好的学习方法是，能够结合实际任务中的用例，仔细阅读背后实现对应Pandas函数。这样在自身遇到类似的情况时，很容易联想到可以使用的Pandas命令。这篇文章收集了在实际工作中常见的Pandas数据清洗用例，每一个用例给出原来的表格式与数据、处理后的表格式与数据，以及相应的pandas代码，在一些函数后，我也附上一些注释，进行一些拓展讲解。比较推荐读者熟悉Pandas基础用法后再进行阅读，入门学习时，官方教学[User Guide](https://pandas.pydata.org/docs/user_guide/index.html)是一个不错的资源。

<!--more-->

<!-- 此外，另外一篇文章(XXXXXXX插入链接)将这篇文章其中的Pandas代码删去，保留剩下的内容，可以将其当成习题册供练习使用。 -->

{{<info>}}
这篇文章还未完成，后面内容和排版还会持续更新。
{{</info>}}

### （入门）加一列，显示每组的平均值

表：

```
>>> df = pd.DataFrame({
    "group": ['a', 'a', 'a', 'b', 'b', 'b'],
    "value": [1, 2, 3, 4, 5, 6]
})
>>> df.set_index("group", inplace=True)
>>> df

       value
group       
a          1
a          2
a          3
b          4
b          5
b          6
```

目标：

```
       value  mean
group             
a          1   2.0
a          2   2.0
a          3   2.0
b          4   5.0
b          5   5.0
b          6   5.0
```

实现方法：

```python
# method 1
df['mean'] = df.groupby(level="group").transform("mean")

# method 2
df['mean'] = df.groupby(level="group").mean()
```

注：

1. 在第2个方法中，我们用到Pandas的一个特性，在赋值时，如果等号后面的变量是带label的（这里的label是group），pandas会将被赋值df中的label与赋值的label进行对齐。


### （入门）加一列，显示组内排名

表：

```
>>> df = pd.DataFrame({
    "type": ['a', 'a', 'b', 'b', 'a', 'a', 'b', 'b'],
    "ammount": [100, 200, 300, 400, 500, 600, 700, 800]
})
>>> df

  type  ammount 
0    a      100     
1    a      200     
2    b      300      
3    b      400   
4    a      500 
5    a      600
6    b      700
7    b      800
```

目标：

```
  type  ammount  group_rank
0    a      100         1.0
1    a      200         2.0
2    b      300         1.0
3    b      400         2.0
4    a      500         3.0
5    a      600         4.0
6    b      700         3.0
7    b      800         4.0
```

实现方法：

```python
df["group_rank"] = df.groupby("type")["value"].rank(method="min", ascending=True)
```

注：

1. rank方法中的method参数控制，在值相同情况下返回排名的方式。其中min返回相等值中最前的排名，相当于SQL中`rank()`；max相等值中最后的排名；average返回相等值中的平均排名；first在值相同的情况下，按照表从上到下顺序继续进行排名，相当于SQL中的`row_number()`，dense表示当前值后一个值的排名相对只加一，相当于SQL中的`dense_rank()`。以 100, 100, 200, 300 为例，从小到大排序，不同参数返回结果如下
  - min [1, 1, 3, 4]
  - max [2, 2, 3, 4]
  - first[1, 2, 3, 4]
  - average [1.5, 1.5, 3, 4]
  - dense [1, 1, 2, 3]



### （入门）加一列，显示累计到现在的平均值

表：

```
>>> col_date = pd.to_datetime(["2023-01-01", "2023-01-02", "2023-01-03", "2023-01-04", "2023-01-05", "2023-01-06", "2023-01-07", "2023-01-08"])
>>> col_ammount = [100, 200, 300, 400, 500, 600, 700, 800]
>>> df = pd.DataFrame({
    "date": col_date,
    "ammount": col_ammount
})
>>> df

        date  ammount
0 2023-01-01      100
1 2023-01-02      200
2 2023-01-03      300
3 2023-01-04      400
4 2023-01-05      500
5 2023-01-06      600
6 2023-01-07      700
7 2023-01-08      800
```

目标：

```
        date  ammount  rolling_mean
0 2023-01-01      100         100.0
1 2023-01-02      200         150.0
2 2023-01-03      300         200.0
3 2023-01-04      400         250.0
4 2023-01-05      500         300.0
5 2023-01-06      600         350.0
6 2023-01-07      700         400.0
7 2023-01-08      800         450.0
```

实现方法：

```python
df["rolling_mean"] = df.expanding()['ammount'].mean()
```


### （入门）加一列，显示组内上一值

表：
```
>>> col_1 = pd.to_datetime(["2023-01-01", "2023-01-02", "2023-01-03", "2023-01-04", "2023-01-05", "2023-01-06", "2023-01-07", "2023-01-08"])
>>> col_2 = ['a', 'a', 'b', 'b', 'a', 'a', 'b', 'b']
>>> col_3 = [200, 200, 300, 400, 500, 600, 700, 800]
>>> df = pd.DataFrame({
    "date": col_1,
    "type": col_2,
    "ammount": col_3
})

        date type  ammount
0 2023-01-01    a      200
1 2023-01-02    a      200
2 2023-01-03    b      300
3 2023-01-04    b      400
4 2023-01-05    a      500
5 2023-01-06    a      600
6 2023-01-07    b      700
7 2023-01-08    b      800
```

目标：

```
        date type  ammount  last_ammount_of_type
0 2023-01-01    a      200                   NaN
1 2023-01-02    a      200                 200.0
2 2023-01-03    b      300                   NaN
3 2023-01-04    b      400                 300.0
4 2023-01-05    a      500                 200.0
5 2023-01-06    a      600                 500.0
6 2023-01-07    b      700                 400.0
7 2023-01-08    b      800                 700.0
```

实现方法：

```python
df['last_ammount_of_type'] = df.groupby("type")['ammount'].shift(periods=1)
```

### （入门）一行展开成多行

表：

```
>>> df = pd.DataFrame({
    "姓名": ["张三", "李四", "王五"],
    "爱好": ["运动,烹饪", "音乐,美术,旅游,影视", "阅读"]
})
>>> df

  Name        Hobby
0   张三        运动,烹饪
1   李四  音乐,美术,旅游,影视
2   王五           阅读
```

目标：

```
  Name Hobby
0   张三    运动
1   张三    烹饪
2   李四    音乐
3   李四    美术
4   李四    旅游
5   李四    影视
6   王五    阅读
```

实现方法：

```python
df.assign(Hobby=df['Hobby'].str.split(",")).explode('Hobby', ignore_index=True)
```

### （入门）多行转多列

表：

```
>>> df = pd.DataFrame({
    "姓名": ["张三", "张三", "张三", "李四", "李四", "李四"],
    "科目": ["语文", "数学", "体育", "语文", "数学", "体育"],
    "成绩": [82, 74, 90, 95, 94, 78]
})

   姓名  科目  成绩
0  张三  语文  82
1  张三  数学  74
2  张三  体育  90
3  李四  语文  95
4  李四  数学  94
5  李四  体育  78
```

目标：

```
      姓名  体育  数学  语文
0   张三  90  74  82
1   李四  78  94  95
```

实现方法：

```python
df.pivot(index="姓名", columns='科目', values="成绩").reset_index()
```

### （入门）多列转多行

```
>>> df = pd.DataFrame({
    ˓名': ['张三', '李四'],
    '语文': [82, 95],
    '数学': [74, 94],
    '体育': [90, 78]
})
>>> df

   姓名  语文  数学  体育
0  张三  82  74  90
1  李四  95  94  78
```

```
   姓名  科目  成绩
0  张三  语文  82
1  李四  语文  95
2  张三  数学  74
3  李四  数学  94
4  张三  体育  90
5  李四  体育  78
```

```python
df.melt(id_vars="姓名", value_vars=["语文", "数学", "体育"], var_name="科目", value_name="成绩")
```


### （入门）两表join

表：
```
>>> df_student_info = pd.DataFrame({
    'student_id': [1, 2, 3],
    'student_name': ['张三', '李四', '王五'],
    'class_number': [5, 7, 8]
})
>>> df_student_info
   student_id student_name  class_number
0           1           张三             5
1           2           李四             7
2           3           王五             8

>>> df_grade = pd.DataFrame({
    'student_id': [1, 2, 3],
    'grade': [82, 95, 85]
})
>>> df_grade

   student_id  grade
0           1     82
1           2     95
2           3     85
```

目标：

```
   student_id student_name  class_number  grade
0           1           张三             5     82
1           2           李四             7     95
2           3           王五             8     85
```

实现方法：

```python
# 实现方法1
pd.merge(df_student_info, df_grade, on='student_id', how='inner')

#实现方法2
df_student_info.merge(df_grade, on='student_id', how='inner')
```


###（入门）选取以某个相同字符开头的列

表：

```
>>> df = pd.DataFrame({
    'name': ['Adam', 'Bob', 'Katty', 'John'],
    'score_1': [39, 72, 73, 84],
    'rank_1': [39, 72, 73, 84],
    'score_2': [92, 49, 87, 62],
    'rank_2': [39, 72, 73, 84],
    'score_3': [86, 75, 90, 45],
    'rank_3': [39, 72, 73, 84]
})
>>> df

    name  score_1  rank_1  score_2  rank_2  score_3  rank_3
0   Adam       39      39       92      39       86      39
1    Bob       72      72       49      72       75      72
2  Katty       73      73       87      73       90      73
3   John       84      84       62      84       45      84
```

目标：

```
   score_1  score_2  score_3
0       39       92       86
1       72       49       75
2       73       87       90
3       84       62       45
```

实现方式： 

```python
df.filter(like='score_', axis=1)
df.filter(regex='^score_', axis=1)
```

注：

1. filter 是按照行或列的label值过滤（通常是字符串label）
2. like 参数保留label有子字符串为入参的行或列
3. regex 参数保留label中有子字符串匹配上入参的行或列


### （入门）选出列表中的数值列

表：
```
>>> df = pd.DataFrame({
"姓名": ["张三", "李四", "王五"],
"爱好": ["音乐", "电影", "跑步"],
“身高”: [178, 163, 186],
“体重": [74, 55, 90]
})
>>> df

   姓名  爱好   身高  体重
0  张三  音乐  178  74
1  李四  电影  163  55
2  王五  跑步  186  90
```

目标：
```
    身高  体重
0  178  74
1  163  55
2  186  90
```

实现方法：

```python
df.select_dtypes("number")
```


### （进阶）用新表更新旧表数据

表：

```
>>> df_1 = pd.DataFrame({
    'date': ['2023-09-01', '2023-09-01', '2023-09-01', '2023-09-01'],
    'name':  ['Adam', 'Bob', 'John', 'Tim'],
    'status': [1, 1, 1, 1]
})
>>> df_2 = pd.DataFrame({
    'date': ['2023-09-02', '2023-09-02'],
    'name':  ['Adam', 'Bob'],
    'status': [2, 2]
})
>>> df_1

         date  name  status
0  2023-09-01  Adam       1
1  2023-09-01   Bob       1
2  2023-09-01  John       1
3  2023-09-01   Tim       1

>>> df_2

         date  name  status
0  2023-09-02  Adam       2
1  2023-09-02   Bob       2
```

目标：

```
         date  name  status
0  2023-09-01  John       1
1  2023-09-01   Tim       1
2  2023-09-02  Adam       2
3  2023-09-02   Bob       2
```

实现方法：
```python
pd.concat([df_1, df_2]).drop_duplicates("name", keep='last', ignore_index=True)
```


### （进阶）按照

源：
```
>>> df = pd.DataFrame([
    (12, 8.7),
    (13, 9.0),
    (14, 8.3),
    (15, 7.3),
    (16, 7.9),
    (25, 6.5),
    (26, 6.9),
    (27, 7.8),
    (28, 7.4),
    (40, 7.0),
    (41, 7.1),
    (42, 7.3)
], columns=['age', "sleep_hour"])
>>> df

    age  sleep_hour
0    12         8.7
1    13         9.0
2    14         8.3
3    15         7.3
4    16         7.9
5    25         6.5
6    26         6.9
7    27         7.8
8    28         7.4
9    40         7.0
10   41         7.1
11   42         7.3
```


目标：

```
            sleep_hour
age_group             
少年(12-18)     8.240000
青年(18-30)     7.150000
中青年(30-40)         NaN
中年(40-50)     7.133333
```


实现方法：

```python
labels = ["少年(12-18)", "青年(18-30)", "中青年(30-40)", "中年(40-50)"]
df['age_group'] = pd.cut(df['age'], bins=[12, 18, 30, 40, 50], labels=labels, right=False)
df.groupby("age_group", observed=False)[['sleep_hour']].mean()
```

注：

1. 当 groupby 一个 category 类型的列时，默认不会打印所有category值，如果该category值下没有对应数据，会被省略掉，设置 observed=False 取消该特性。


### （进阶）用组内平均值填充

源：

```
>>> df = pd.DataFrame([
    {
        "name": "Brenda",
        "region": "北京",
        "salary": 29
    },
    {
        "name": "Caleb",
        "region": "北京",
        "salary": None
    },
    {
        "name": "Lael",
        "region": "北京",
        "salary": None
    },
    {
        "name": "Jordan",
        "region": "北京",
        "salary": 24
    },
    {
        "name": "Troy",
        "region": "上海",
        "salary": 19
    },
    {
        "name": "Justin",
        "region": "上海",
        "salary": 20
    },
    {
        "name": "Xenos",
        "region": "上海",
        "salary": 27
    },
    {
        "name": "Maya",
        "region": "上海",
        "salary": 27
    },
    {
        "name": "Dorian",
        "region": "深圳",
        "salary": 21
    },
    {
        "name": "Rhona",
        "region": "深圳",
        "salary": 30
    },
    {
        "name": "Solomon",
        "region": "深圳",
        "salary": None
    },
    {
        "name": "Martena",
        "region": "深圳",
        "salary": 20
    }
])
>>> df

       name region  salary
0    Brenda     北京    29.0
1     Caleb     北京     NaN
2      Lael     北京     NaN
3    Jordan     北京    24.0
4      Troy     上海    19.0
5    Justin     上海    20.0
6     Xenos     上海    27.0
7      Maya     上海    27.0
8    Dorian     深圳    21.0
9     Rhona     深圳    30.0
10  Solomon     深圳     NaN
11  Martena     深圳    20.0
```

目标：

```
       name region  salary
0    Brenda     北京   29.00
1     Caleb     北京   26.50
2      Lael     北京   26.50
3    Jordan     北京   24.00
4      Troy     上海   19.00
5    Justin     上海   20.00
6     Xenos     上海   27.00
7      Maya     上海   27.00
8    Dorian     深圳   21.00
9     Rhona     深圳   30.00
10  Solomon     深圳   23.67
11  Martena     深圳   20.00
```

实现方法：

```python
_numeric_columns = df.select_dtypes('number').columns
mean_to_fill = df.groupby("region")[_numeric_columns].transform("mean")
df.fillna(mean_to_fill).round(2)
```

注：

1. `df.select_dtypes('number').columns` 返回所有的数值列


### （进阶）对一个表不同列使用不同的聚合函数

```
>>> df = pd.DataFrame([
    {
        "name": "Brenda",
        "region": "北京",
        "salary": 29
    },
    {
        "name": "Caleb",
        "region": "北京",
        "salary": None
    },
    {
        "name": "Lael",
        "region": "北京",
        "salary": None
    },
    {
        "name": "Jordan",
        "region": "北京",
        "salary": 24
    },
    {
        "name": "Troy",
        "region": "上海",
        "salary": 19
    },
    {
        "name": "Justin",
        "region": "上海",
        "salary": 20
    },
    {
        "name": "Xenos",
        "region": "上海",
        "salary": 27
    },
    {
        "name": "Maya",
        "region": "上海",
        "salary": 27
    },
    {
        "name": "Dorian",
        "region": "深圳",
        "salary": 21
    },
    {
        "name": "Rhona",
        "region": "深圳",
        "salary": 30
    },
    {
        "name": "Solomon",
        "region": "深圳",
        "salary": None
    },
    {
        "name": "Martena",
        "region": "深圳",
        "salary": 20
    }
])
>>> df

       name region  salary
0    Brenda     北京    29.0
1     Caleb     北京     NaN
2      Lael     北京     NaN
3    Jordan     北京    24.0
4      Troy     上海    19.0
5    Justin     上海    20.0
6     Xenos     上海    27.0
7      Maya     上海    27.0
8    Dorian     深圳    21.0
9     Rhona     深圳    30.0
10  Solomon     深圳     NaN
11  Martena     深圳    20.0
```

目标：
```
        people_count  salary_mean
region                           
上海                 4        23.25
北京                 4        26.50
深圳                 4        23.67
```

实现方法：

```python
df.groupby("region").agg(
    people_count=pd.NamedAgg(column="name", aggfunc="count"),
    salary_mean=pd.NamedAgg(column="salary", aggfunc="mean"),
).round(2)
```

注
1. 在计算mean时，会将空值剔除。（计算count时候也会将空值剔除，不过这里没有体现出来）


### （进阶）保留数据量大于3的组

源：

```
>>> df = pd.DataFrame({
    'name': ['Adam', 'Bob', 'Jack', 'John', 'Mike', 'David'],
    'group': ['A', 'A', 'A', 'A', 'B', 'B']
})
>>> df

    name group
0   Adam     A
1    Bob     A
2   Jack     A
3   John     A
4   Mike     B
5  David     B
```

目标：

```
   name group
0  Adam     A
1   Bob     A
2  Jack     A
3  John     A
```

实现方法：

```python
df.groupby("group").filter(lambda x: len(x) > 3)
```

### （入门）生成dummy variable

表：

```
>>> df = pd.DataFrame({
    "name": ['Adam', 'Bob', 'John'],
    "hobby": ['Music', 'Music', 'Sports'] 
})
>>> df

   name   hobby
0  Adam   Music
1   Bob   Music
2  John  Sports
```

目标：

```
   name  hobby_Music  hobby_Sports
0  Adam            1             0
1   Bob            1             0
2  John            0             1
```

实现方法：

```python
_dummy_table = pd.get_dummies(df['hobby'], prefix="hobby").astype('int')
a(pd.concat([df['name'], _dummy_table], axis=1))
```

### dummy variable 转换

表：
```
>>> df = pd.DataFrame({
"name": ["Adam", "Bob", "John"],
"hobby_Music": [1, 1, 0],
"hobby_Sports": [0, 0, 1]
})
>>> df
   name  hobby_Music  hobby_Sports
0  Adam            1             0
1   Bob            1             0
2  John            0             1
```

目标：
```
   name   hobby
0  Adam   Music
1   Bob   Music
2  John  Sports
```

实现方法：
```python
_column = pd.from_dummies(df.filter(like="hobby_", axis=1), sep="_")
pd.concat([df['name'], _column], axis=1)
```


### （入门）更改表列名

表：
```
>>> df = pd.DataFrame({
    "col_1": ['Adam', 'Bob', 'John'],
    "col_2": ['Music', 'Music', 'Sports'] 
})
>>> df

  col_1   col_2
0  Adam   Music
1   Bob   Music
2  John  Sports
```

目标：
```
   name   hobby
0  Adam   Music
1   Bob   Music
2  John  Sports
```

实现方法：
```python
df.rename(columns={"col_1": "name", "col_2": "hobby"})
```


### （入门）删除表的某一列

表：

```
>>> df = pd.DataFrame({
    'date': ['2023-09-01', '2023-09-01', '2023-09-01', '2023-09-01'],
    'name':  ['Adam', 'Bob', 'John', 'Tim'],
    'status': [1, 1, 1, 1]
})
>>> df

         date  name  status
0  2023-09-01  Adam       1
1  2023-09-01   Bob       1
2  2023-09-01  John       1
3  2023-09-01   Tim       1

```

目标：
```
         date  name
0  2023-09-01  Adam
1  2023-09-01   Bob
2  2023-09-01  John
3  2023-09-01   Tim

```

实现方法：
```python
df.drop(columns=['status'])
```

### （进阶）表多行转多列

表：

```
>>> df = pd.DataFrame({
    "姓名": ["张三", "张三", "张三", "李四", "李四", "李四"],
    "科目": ["语文", "数学", "体育", "语文", "数学", "体育"],
    "成绩": [82, 74, 90, 95, 94, 78]
})

   姓名  科目  成绩
0  张三  语文  82
1  张三  数学  74
2  张三  体育  90
3  李四  语文  95
4  李四  数学  94
5  李四  体育  78
```

目标：

```
      姓名  体育  数学  语文
0   张三  90  74  82
1   李四  78  94  95
```

实现方法：

```python
df.pivot(index="姓名", columns='科目', values="成绩").reset_index()
```

### （进阶）表多列行转多行

表：

```
>>> df = pd.DataFrame({
    '姓名': ['张三', '李四'],
    '语文': [82, 95],
    '数学': [74, 94],
    '体育': [90, 78]
})
>>> df

   姓名  语文  数学  体育
0  张三  82  74  90
1  李四  95  94  78
```

目标：

```
   姓名  科目  成绩
0  张三  语文  82
1  李四  语文  95
2  张三  数学  74
3  李四  数学  94
4  张三  体育  90
5  李四  体育  78
```

实现方法：

```python
df.melt(id_vars="姓名", value_vars=["语文", "数学", "体育"], var_name="科目", value_name="成绩")
```

### （入门）选取列为某些值的行

表：
```
>>> df = pd.DataFrame({
    'name':  ['Adam', 'Bob', 'John', 'Tim', 'Jack', 'Lucas', 'White', 'Frank'],
    'height': [162, 180, 173, 194, 177, 170, 172, 169]
})
>>> df

    name  height
0   Adam     162
1    Bob     180
2   John     173
3    Tim     194
4   Jack     177
5  Lucas     170
6  White     172
7  Frank     169
```

目标：
```
    name  height
0   Adam     162
1    Bob     180
6  White     172
7  Frank     169
```

实现方法：
```python
df[df['name'].isin(["Adam", "Bob", "White", "Frank"])]
```

### （进阶）每组排名第2和第3的行


表：

```
>>> df = pd.DataFrame([
    {
        "name": "Brenda",
        "region": "北京",
        "salary": 29
    },
    {
        "name": "Caleb",
        "region": "北京",
        "salary": 25
    },
    {
        "name": "Lael",
        "region": "北京",
        "salary": 20
    },
    {
        "name": "Jordan",
        "region": "北京",
        "salary": 24
    },
    {
        "name": "Troy",
        "region": "上海",
        "salary": 19
    },
    {
        "name": "Justin",
        "region": "上海",
        "salary": 20
    },
    {
        "name": "Xenos",
        "region": "上海",
        "salary": 27
    },
    {
        "name": "Maya",
        "region": "上海",
        "salary": 27
    },
    {
        "name": "Dorian",
        "region": "深圳",
        "salary": 21
    },
    {
        "name": "Rhona",
        "region": "深圳",
        "salary": 30
    },
    {
        "name": "Solomon",
        "region": "深圳",
        "salary": 18
    },
    {
        "name": "Martena",
        "region": "深圳",
        "salary": 20
    }
])

>>> df
       name region  salary
0    Brenda     北京      29
1     Caleb     北京      25
2      Lael     北京      20
3    Jordan     北京      24
4      Troy     上海      19
5    Justin     上海      20
6     Xenos     上海      27
7      Maya     上海      27
8    Dorian     深圳      21
9     Rhona     深圳      30
10  Solomon     深圳      18
11  Martena     深圳      20
```

目标：

（取每个地区工资排第2和第3的数据）

```
          name  salary
region                
上海       Xenos      27
上海        Maya      27
北京       Caleb      25
北京      Brenda      29
深圳      Dorian      21
深圳       Rhona      30
```

实现方法：

```python
df.sort_values('salary').groupby("region").nth([2, 3])
```


### （进阶）用户连续登陆的天数

表：

```
>>> df = pd.DataFrame([
    ('adam', '2023-01-01'),
    ('adam', '2023-01-02'),
    ('bob', '2023-01-02'),
    ('adam', '2023-01-03'),
    ('bob', '2023-01-04'),
    ('adam', '2023-01-05'),
    ('bob', '2023-01-05'),
], columns=['user', 'login_date'])
>>> df['login_date'] = pd.to_datetime(df['login_date'])
>>> df

   user login_date
0  adam 2023-01-01
1  adam 2023-01-02
2   bob 2023-01-02
3  adam 2023-01-03
4   bob 2023-01-04
5  adam 2023-01-05
6   bob 2023-01-05
```

目标：

```
   user login_date  consecutive_login_day
0  adam 2023-01-01                      1
1  adam 2023-01-02                      2
2   bob 2023-01-02                      1
3  adam 2023-01-03                      3
4   bob 2023-01-04                      1
5  adam 2023-01-05                      1
6   bob 2023-01-05                      2
```

实现方法：

```python
def get_consecutive_day(df):
    date_rank_within_user = df.sort_values('login_date').groupby(['user'])['login_date'].rank()
    df_with_start_date = df.assign(consecutive_start_date = df['login_date'] - pd.to_timedelta(date_rank_within_user, unit='D'))
    return df.assign(consecutive_login_day=df_with_start_date.groupby(['user', 'consecutive_start_date']).rank().astype(int))

df.pipe(get_consecutive_day)
```



