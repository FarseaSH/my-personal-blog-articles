---
date: 2023-06-12T11:04:00+08:00
title: Pandas 选取行、选取列方式梳理
category: 想法与笔记
tags: 
 - Python
 - Pandas
toc: true
toc_fold: true
updated: 
copyright: true  # 暂不支持
draft: false
---

这篇笔记对Pandas DataFrame取数的各种写法与对应的各种返回结果做一个梳理。

如没有特殊声明，df变量的数据如下：

```
>>> df

   col_1 col_2
a      0   foo
b      1   foo
c      2   bar
d      3   bar
e      4   foo
f      5   foo
g      6   bar
h      7   bar
```

<!--more-->

## df.loc[] 基于标签的筛选

### single label 单一标签

输入为某个单一行标签label时，返回label所对应单一的行，返回类型为Series（如果行标签与行一一对应）。

```
>>> df.loc['a']

col_1      0
col_2    foo
```

注1：如果没有对应参数的label，则报KeyError。

```
>>> df.loc['z']

KeyError: 'z'
```

注2：如果index中label不唯一，多个行有相同的行标签时，则返回所有有该标签的行，返回类型为DataFrame，如：

```
>>> df_duplicate_index

   col_1 col_2
a      0   foo
a      1   foo
b      2   bar

>>> df_duplicate_index.loc['a']

   col_1 col_2
a      0   foo
a      1   foo
```

### list of labels 标签

输入为一个行标签list时，按照list中顺序，返回这些标签对应的所有行，返回类型为DataFrame。

```
>>> df.loc[['a', 'b', 'c']]

   col_1 col_2
a      0   foo
b      1   foo
c      2   bar
```

注1：如果list里没有对应参数的标签，则报KeyError。

```
>>> df.loc[['a', 'b', 'z']]

KeyError: "['z'] not in index"
```

注2：与单一标签一致的，如果index中label不唯一时，则所有带入参标签的行都会被返回。

注3：loc入参list中的label是可以重复的，通过这样来得到重复选取某行的返回结果。

```
>>> df.loc[['a', 'a', 'a']]

   col_1 col_2
a      0   foo
a      0   foo
a      0   foo
```

### slice object 切片对象

slice 对象是指Python中带冒号的表达式，如1:5。返回类型为DataFrame。

返回结果的需要根据index是否有序，分情况讨论：

- 当index是有序的，slice中的参数无需为index中某一个label，返回label在slice区间内的行，返回类型为DataFrame。
- 当index是无序的，slice中的参数**必须**为index中某个label，否则会报KeyError，返回label为slice参数中的行，以及这两行中间的行，返回类型为DataFrame。

```
>>> df.loc["a":"e"]

   col_1 col_2
a      0   foo
b      1   foo
c      2   bar
d      3   bar
e      4   foo

>>> df.loc["a":"z"]  # 注意z不在index里

   col_1 col_2
a      0   foo
b      1   foo
c      2   bar
d      3   bar
e      4   foo
f      5   foo
g      6   bar
h      7   bar

>>> df_unsorted_index = df.loc[['f', 'a', 'c', 'g', 'h', 'd', 'e', 'b']]
>>> df_unsorted_index.loc["a":"e"]

   col_1 col_2
a      0   foo
c      2   bar
g      6   bar
h      7   bar
d      3   bar
e      4   foo
```

注1：loc中slice两端端点的处理与Python中slice有些不同，python中slice是“左开右闭”的，即冒号右边的值不会囊括，如 `[i for i in range(10)][0:3]` 返回 [0, 1, 2]，而在loc中slice是**both inclusive**的，意味着冒号前后的值都会被囊括。

注2：可以用`df.index.is_monotonic_increasing`或 `df.index.is_monotonic_decreasing`（是一个属性非方法，不带括号）来判断dataframe是否已经按照index排好序。

注3：如果slice中冒号前一个label对应行在冒号后一个label对应行的后面，则会返回一个空的DataFrame。

```
>>> df.loc["e":"a"]

Empty DataFrame
Columns: [col_1, col_2]
Index: []
```

注4：当index中label不唯一时，使用slice可能会报错。

```
>>> df_duplicate_index_2

   col_1 col_2
a      0   foo
b      1   foo
c      2   bar
a      3   bar

>>> df_duplicate_index_2["a":"b"]

KeyError: "Cannot get left slice bound for non-unique label: 'a'"
```

### boolean array 布尔列表

输入为一个数量与表行数相同（必须相同）的布尔列表时，返回对应位置为True的行，返回类型为DataFrame。

这种调用方式常用于按照某个条件过滤dataFrame。

```
>>> df.loc[df["col_2"] == "foo"]

   col_1 col_2
a      0   foo
b      1   foo
e      4   foo
f      5   foo
```

注1：条件的逻辑运算或与非需要用位运算符`&`, `|` , `~`，在进行两个条件运算时，因为Python运算优先级，需要将条件语句用圆括号括起来。

```
>>> df.loc[df["col_1"] > 1 & df["col_2"] == "foo"] #直接做位运算会报错

TypeError: Cannot perform 'rand_' with a dtyped [object] array and scalar of type [bool]

>>> df.loc[(df["col_1"] > 1) & (df["col_2"] == "foo")]      #将条件语句括起来提高运算优先级

   col_1 col_2
e      4   foo
f      5   foo
```

### callable 函数

输入为一个函数时，会将dataframe传入函数的返回结果作为loc的入参，返回对应的结果。函数的返回类型可以为上面提到4种（单一label、label列表、slice对象、布尔列表）中的任意一种。

函数入参主要是为了支持链式写法，当我们需要多重筛选dataframe时，使用链式写法进行代码开发不需要存储中间的dataframe结果，使得我们的代码更加简洁。

```python
# 常规写法
df_mid_1 = df.loc[df["col_1"] > 1]
df_mid_2 = df_mid_1.loc[df_mid_1.groupby("col_2")["col_2"].transform("count") > 2]
df_mid_2.loc[df_mid_2.index.isin(["a", "d", "g"])]

# 链式写法
(
    df.loc[df["col_1"] > 1]
        .loc[lambda x: x.groupby("col_2")["col_2"].transform("count") > 2]
        .loc[lambda x: x.index.isin(["a", "d", "g"])]
)
```

## df.iloc[] 基于行号的筛选

iloc与loc最大的不同在于，loc接受的参数是index中的label，而iloc接受的参数是整数，代表dataframe的行号（从0开始）。

### single integer 单一整数

输入为单一行数时，返回代数所在单一的行，返回类型为Series。

```
>>> df.iloc[0]

col_1      0
col_2    foo
```

注1：入参的整数可以为负数，会返回倒数第x行。

```
>>> df.iloc[-1] 

col_1      7 col_2    bar
```
注2：如果整数超过了dataframe的总行数，则报IndexError。

```
>>> df.iloc[10]

IndexError: single positional indexer is out-of-bounds
```

### list of labels 整数列表

输入为一个整数list时，按照list中顺序，返回这些行数对应的所有行，返回类型为DataFrame。

```
>>> df.iloc[[0, 1, 2]]

   col_1 col_2
a      0   foo
b      1   foo
c      2   bar
```

注1：如果list里面的整数超过了dataframe的总行数，则报IndexError。

```
>>> (df.iloc[[0, 10]]

IndexError: positional indexers are out-of-bounds
```

注2：iloc入参list中的整数是可以重复的，通过这样来得到重复选取某行的返回结果。

```
>>> df.iloc[[0, 0, 0]]

   col_1 col_2
a      0   foo
a      0   foo
a      0   foo
```

### slice object 切片对象

输入为一个整数slice时，返回slice对应的行数（左闭右开，只包含冒号前面的行），返回类型为DataFrame。

```
>>> df.iloc[1:5]

   col_1 col_2
b      1   foo
c      2   bar
d      3   bar
e      4   foo
```

注1：需要注意的是，与loc slice入参冒号前后的label都会被返回不同，iloc中冒号后面的整数对应的行不会被返回。

```
>>> df_int_index = pd.DataFrame(["a", "b", "c"], columns=["col"], index=[0, 1, 2])
>>> df_int_index.iloc[0:2]

  col
0   a
1   b

>>> df_int_index.loc[0:2]

  col
0   a
1   b
2   c
```

### boolean array 布尔列表

与loc中 [boolean array 布尔列表](/2023/06/12/pandas_loc/#boolean-array-%E5%B8%83%E5%B0%94%E5%88%97%E8%A1%A8)用法完全相同。

### callalbe 函数

与loc中[callable函数](/2023/06/12/pandas_loc/#callable-%E5%87%BD%E6%95%B0)用法基本相同，唯一不同是函数的返回类型需要为单一整数、整数列表、整数slice、布尔列表四种中的一种。

## df[] 筛选

当直接在df[]输入筛选语句时，需要注意输入的参数类型对应的是筛选行还是筛选列，具体情况如下：

### single label 单一标签 - 返回列

输入为某个单一标签label，返回label所对应的**列**，返回类型为Series（如果列标签与列一一对应）。`df["a"]`等价于`df.loc[:, "a"]`

```
>>> df['col_1']

a    0
b    1
c    2
d    3
e    4
f    5
g    6
h    7

>>> df['col_1'].equals(df.loc[:, 'col_1'])

True
```

### list of labels 标签 - 返回列

输入为一个标签list时，按照list中顺序，返回这些标签对应的所有**列**，返回类型为DataFrame，`df[['col_2', 'col_1']]` 等价于 `df.loc[:, ['col_2', 'col_1']]`

```
>>> df[['col_2', 'col_1']]

  col_2  col_1
a   foo      0
b   foo      1
c   bar      2
d   bar      3
e   foo      4
f   foo      5
g   bar      6
h   bar      7

>>> df[['col_2', 'col_1']].equals(df.loc[:, ['col_2', 'col_1']])

True
```

### label slice 标签的切片对象 - 返回行

输入为一个 label slice 标签时(e.g. `df["d":"g"]`)，返回slice对应的**行**，返回类型为DataFrame。`df["d":"g"]` 等价于 `df.loc["d":"g"]`，具体逻辑可以参考 `loc` 部分的[slice-object-切片对象](/2023/06/12/pandas_loc/#slice-object-切片对象)

```
>>> df["d":"g"]

   col_1 col_2
d      3   bar
e      4   foo
f      5   foo
g      6   bar

>>> df["d":"g"].equals(df.loc["d":"g"])

True
```

### integer slice 整数的切片对象 - 返回行

输入为一个整数slice(e.g. `df[1:5]`) 时，返回slice对应行数所在的**行**，返回类型为DataFrame。`df[1:5]` 等价于 `df.iloc[1:5]`，具体逻辑可以参考 `.iloc` 部分的[slice-object-切片对象](/2023/06/12/pandas_loc/#slice-object-切片对象-1)
```
>>> df[2:5]

   col_1 col_2
c      2   bar
d      3   bar
e      4   foo

>>> df[2:5].equals(df.iloc[2:5])

True
```

### boolean array 布尔列表 - 返回行

输入为时，返回对应位置为True的**行**(布尔列表长度需与dataFrame行数一致)，返回类型为DataFrame。`df[df['col_1'] > 3]` 等价于 `df.loc[df['col_1'] > 3]`

```
>>> df[df['col_1'] > 3]

   col_1 col_2
e      4   foo
f      5   foo
g      6   bar
h      7   bar

>>> df[df['col_1'] > 3].equals(df.loc[df['col_1'] > 3])

True
```

### callable 函数

我们可以传一个函数，与loc iloc情况类似，会将dataframe传入函数的返回结果作为loc的入参，返回对应的结果。

```
>>> df[lambda x: x['col_1'] > 3]

   col_1 col_2
e      4   foo
f      5   foo
g      6   bar
h      7   bar
```

## 其它

### 整数与label的混用

在筛选行于列一同筛选时，有时候为了方便，在筛选行时用输入整数行数进行选取，在筛选列时输入label进行选取。loc与iloc是不允许同时输入两种类型的，但我们可以借助一些辅助函数完成两种类型的混用。

在`.iloc[]`中，使用`dataFrame.columns.get_loc()`（单一label）或 `dataFrame.columns.get_indexer([])`（label列表），将label转换为对应的列数。

```
>>> df.iloc[[0, 1], [df.columns.get_loc('col_1')]]

   col_1
a      0
b      1
```

在`.loc[]`中，使用`dataFrame.index[[]]`，将行数转换为对应的label。

```
>>> df.loc[df.index[[0, 1]], ['col_1']

   col_1
a      0
b      1
```

注意：如果一个lable对应多个行，`dataFrame.index[[]]`可能会返回某行数label对应的所有的行。
