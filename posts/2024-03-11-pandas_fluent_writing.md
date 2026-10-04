---
title: Pandas链式代码书写
date: 2024-03-11T21:37:00+08:00
updated:
copyright: true  # 暂不支持
category: 经验
tags:
    - Pandas
    - Python
toc: true
toc_fold: true
draft: false
---

[Pandas](https://pandas.pydata.org/) 是Python中最常用的数据分析包，不论是在学校完成课程作业与项目，还是在职场数据相关工作中，都大概率会有所接触 Pandas。绝大部分人学习接触Pandas时以及后续使用Pandas所用的命令都是 inplace 类型的计算，例如添加列使用 `df['new_col'] = df['old_col'] + 1` ，排序使用 `df.sort_values('sort_by_col', inplace=True)`，这些计算直接在原表上操作，代码书写起来比较符合直觉。但在一些链路较长、逻辑复杂、分支较多的数据清洗/分析任务上，这种写法可能出现一些潜在的问题，加大我们代码书写的难度。除了inplace的操作，Pandas所提供的api支持链式代码书写，可以大大增加我们的代码质量，帮助分析师从代码实现的难度中解放出来。本文后面的内容对Pandas链式书写的优势以及相关写法进行一些介绍。

<!--more-->

## 1 什么是Pandas链式代码书写？

大部分Pandas DataFrame的调用方法，执行后会返回一个新的DataFrame。我们可以直接在这个返回的DataFrame上继续进行方法调用，无需给过程中间生成的DataFrame赋值给某个python变量。以此类推，可以在一个 DataFrame 上进行一系列方法操作，一个方法接在上一个方法生成的DataFrame后面，最后得到经过一系列操作后新的的DataFrame。这种代码的写法是一环接一环的，通过一个DataFrame对象加一系列的DataFrame方法得到最后的结果。另外，这个最后结果并非直接在原DataFrame进行修改后得到，而是一个新的DataFrame，原DataFrame的数据是没有变化的。下面的例子可以直观看出两种写法的不同。

```python
# 常规写法
some_df.loc[some_df['col_a'] > 1, 'col_b'] = 'ok'
some_df = some_df[some_df['col_a' < 10]]
some_df.sort_values(by='col_a', inplace=True)
some_df.columns = ['number_col', 'string_col']

# 链式写法
(
    some_df.assign(col_a=lambda df: df['col_b'].mask(df['col_a'] > 1, 'col_b'))
        .loc[lambda df: df['col_a'] < 10]
        .sort_values(by='col_a')
        .rename(columns={'col_a': 'numer_col', 'col_b': 'string_col'})
)

```

## 2 链式写法的优势

想要了解链式写法的优势，我们可以结合常规写法的缺点来进行说明

### 2.1 底层表的修改可能会导致上层数据变化情况难以判断

```python
# 常规写法
df1 = pd.DataFrame([0, 0, 0, 0, 0])
df2 = df1
df1.iloc[:, 0] = 1

# >>> df1
#    val
# 0    1
# 1    1
# 2    1
# 3    1
# 4    1

# >>> df2
#    val
# 0    1
# 1    1
# 2    1
# 3    1
# 4    1
```

考虑上面这个简化的例子，当修改df1时，df2是否会跟随变化？这是一个看起来非常含糊，且无法直接判断的问题。

相应的，链式写法总会返回新表，这些新表是在原表基础上叠加一些操作生成的，并不会影响原表的数据。

```python
# 链式写法
df2 = df1
df1 = df1.assign(val=1)

# >>> df1
#    val
# 0    1
# 1    1
# 2    1
# 3    1
# 4    1

# >>> df2
#    val
# 0    0
# 1    0
# 2    0
# 3    0
# 4    0
```

### 2.2 代码需要修改时，常规写法可能会导致数据的重新加载

```python
# 常规写法
df = pd.read_excel(…)    # 加载一个很大文件

df = …
#  (一系列计算)
df = …

# 下面两种实际开发中常见的写法，对某一列赋值都覆盖了原有的值
# 如果下面这步开发出错，可能需要从头运行上面所有代码
df.loc[:, 'some_col'] = 'some_val'
df['other_col'] = df['other_col'] * 2
```

在Pandas代码开发过程中，当发现某一步写错时，如果是在原有表上数据直接做修改，该操作是无法撤回复原的，我们需要从数据载入处重新开始，执行之前的所有命令，这非常繁琐且耗时，十分影响开发效率。

相对应的，在链式写法中，因为计算操作生成了新表，原表的数据保持不变，我们可以直接修改代码，并从中间表重新执行计算操作，无需再从头载入开始运行代码。

### 2.3 规模清洗任务，常规写法可能会导致较差的代码质量

考虑前一章的例子，想象一下如果采用常规写法开发任务量很大的数据需求，一次清洗任务代码量超过500行，那么这段代码的后期维护会变得非常困难。

这时如果考虑链式写法，代码的格式会变得工整，没有重复的df变量名，每一行表示一个清晰的运算，整个代码的可读性大大提高，后期维护难度大大减小。

### 2.4 常规书写方法，对于代码的复用不太友好

```python
some_df = some_df.sort_values(by=['a', 'b'])
some_df['c'] = 100
# (…其它逻辑...)
some_df['d'] = some_df['a'] - 1
some_df = some_df[some_df['a'] > 3]

# 如果有一个 other_df 想要复用相同的开发逻辑，需要把上面 some_df 全部替换成 other_df

other_df = other_df.sort_values(by=['a', 'b'])
other_df['c'] = 100
# (…其它逻辑...)
other_df['d'] = other_df['a'] - 1
other_df = other_df[other_df['a'] > 3]

(
    some_df
        .sort_values(by=['a', 'b'])
        .assign(c=100)
        # (...其它逻辑...)
        .assign(d=lambda df: df['a'] - 1)
        .loc[lambda df: df['a' > 3, :]
)

# 链式写法直接替换开头的 some_df 即可

(
    other_df
        .sort_values(by=['a', 'b'])
        .assign(c=100)
        # (...其它逻辑...)
        .assign(d=lambda df: df['a'] - 1)
        .loc[lambda df: df['a' > 3, :]
)
```

在常规的Pandas写法中，如果想要复用之前写过的代码，修改起来可能会比较麻烦，但如果使用链式开发，只需要修改开头的DataFrame即可，后续的计算操作无需任何变化。

注：对于需要复用的DataFrame一系列计算的代码，可以封装成一个函数，并使用`pipe`方法传入该函数调用，具体可以参考 [pandas.DataFrame.pipe](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.pipe.html)

## 3 具体的链式写法

### 3.1 新增 / 修改列

大部分Pandas函数都离不开对列的操作，在链式写法中，我们使用`.assign`方法新增列或修改已存在的列。

```python
# 常规写法
some_df['col_c'] = some_df['col_a'] * 2  # 新增一列
some_df['col_b'] = 100  # 修改原有列

# 链式写法
some_df.assign(col_c=some_df['col_a'] * 2)
some_df.assign(**{'col_b': 100})  # 使用字典传入
```

注：因为Python函数参数无法使用中文，所以在编辑/新增中文列名的列数据时，需要使用字典配合两个星号**传入可变参数的形式传入整个assign方法中。

### 3.2 条件赋值

有时候我们需要将符合条件的某些行的某个字段进行修改，可以如下操作：

```python
# 常规写法
some_df.loc[some_df['col_a'] > 5, "col_b"] = 999

# 链式写法
some_df.assign(col_a=some_df['col_a'].mask(some_df['col_b'] > 5, 999))
some_df.assign(col_a=lambda df: df['col_a'].mask(df['col_b'] > 5, 999))   # 使用匿名函数

```

### 3.3 排序去重等常规DataFrame方法

Pandas对表操作的方法印象中都是直接返回一个DataFrame，也即原生设计时候就是为了支持链式写法的，因此在进行链式写法开发时，将inplace参数设置的True值去掉就好了（参数默认为False）。

```python
# 常规写法
some_df.sort_values('col_a', inplace=True)  # 常规写法1
some_df = some_df.sort_values('col_a')  # 常规写法2

some_df.drop_duplicates('col_b', inplace=True)
some_df = some_df.drop_duplicates('col_b')  # 常规写法2

# 链式写法
some_df.sort_values('col_a')
some_df.drop_duplicates('col_b')

```

### 3.4 切片操作

对于筛选行，选中某类列等表切片的操作，我们可以使用`.loc[]`方法，类似上面所说的DataFrame对表操作的函数，`.loc[]`方法也是返回一个DataFrame，形式上已经支持链式写法。

```python
some_df.loc[some_df['col_a'] > 10, :]  # 链式写法 - 条件筛选行
some_df.loc[lambda df: df['col_a'] > 10, :]  # 链式写法 - 使用匿名函数条件，筛选行
some_df.loc[:, ['col_a', 'col_b']] # 链式写法 - 筛选列

```

注

1. loc方法后面应该跟中括号而非圆括号
2. 当选中所有行时，可以省略不写明需要选中的列，但为了可读性，建议将选中的行和列一并写明（使用单一冒号 `:` 选中所有行或列），如 `some_df.loc[lambda df: df['col_a'] > 5]` -> `some_df.loc[lambda df: df['col_a'] > 5, :]`
3. 自己写了一篇文章总结Pandas表切片操作，详情见[Pandas 选取行、选取列方式梳理](https://www.yyshao.com/2023/06/12/pandas_loc/)

### 3.5 函数式写法`lambda df: …`的说明

在前面的例子中我们看到，链式写法中可以传入一个Python匿名函数表达式(即`lambda df: ...`)。初看这个语法可能会令人费解，但实际上很好理解，可以看下面的例子

```python
# 不使用函数表达式
some_df_with_new_col = some_df.assign(c=some_df['a'] + 1)
some_df_final = some_df_with_new_col.loc[some_df_with_new_col['c'] > 5]

# 使用函数表达式
some_df_final = (
    some_df.assign(c=some_df['a'] + 1)
        .loc[lambda df: df['c'] > 5]
)
```

因为链式写法是由一个DataFrame + 一系列计算方法构成，不存在中间的DataFrame，如果有一步计算需要使用中间的DataFrame的数据（像上面例子中的，按新加的`new_col`条件过滤行）就需要将前一步的DataFrame存成变量，再引用这个变量的数据，这显然非常繁琐且失去链式写法的优势。函数表示式作为入参就是用于这种情况，我依旧可以采用链式写法，将需要引用的中间DataFrame数据，替换成一个函数表达式（如上面的，`some_df_with_new_col['c'] > 5` -> `lambda df: df['c'] > 5`），Pandas在运行这行代码时，会自动将上一步的DataFame代入该函数，并将函数返回的结果（也即中间DataFrame的真实数据）传入链式计算这一步方法中，达到引用中间表数据的效果。


## 4 Vscode 快捷输入代码片段

为了进一步提升链式书写Pandas的效率，我将常用的链式代码整理成Vscode代码片段，并将其映射快捷输入命令，让开发的流程更为顺利。

1. 选中某些列
    - 快捷命令: ` .lcc`
    - 记忆: **.L**o**C** for **C**olumn
    - 展开代码: `.loc[:, ['']]`
2. 选中某些行
    - 快捷命令: ` .lcr`
    - 记忆:  **.L**o**C** for **R**ow
    - 展开代码: `.loc[, :]`
3. 创建/修改某些列的值
    - 快捷命令: `.as`
    - 记忆: **.AS**sign
    - 展开代码: `.assign(**{'':})`
4. 排序
    - 快捷命令: `.sv`
    - 记忆: **.S**ort\_**V**alues
    - 展开代码: `.sort_values(by=[''], ascending=[])`
5. 删除某些列
    - 快捷命令: `.dc`
    - 记忆: **.D**rop(, **C**olumns=)
    - 展开代码: `.drop(columns=[''])
6. 行去重
    - 快捷命令: `.dd`
    - 记忆: **.D**rop\_**D**uplicates()
    - 展开代码: `.drop_duplicates([''], keep='first')`
7. 对行应用自定义函数
    - 快捷命令: `.ap1`
    - 记忆: **.A**pply(…, axis=*1*)
    - 展开代码: `.apply(, axis=1)`
8. 列重命名
    - 快捷命令: `.rn`
    - 记忆: .**RE**name
    - 展开代码：`.rename(columns={'': ''}, errors='raise')`
9. 匿名函数
    - 快捷命令: `ldf`
    - 记忆: **L**ambda: **DF**:
    - 展开代码: `lambda df: `


将下面整理好的配置文件放入vscode编辑器即可，具体操作为：打开Configure User Snippets 创建 New Global Snippet，将上面文本全部复制，覆盖整个文件即可。

```json
{
    ".loc[:, ['col']]": {
        "prefix": ".lcc",
        "body": [
            ".loc[:, ['$0']]"
        ],
        "description": ".loc[:, ['col']]"
    },
    ".loc[lambda df: df['a'] > 0, :]": {
        "prefix": ".lcr",
        "body": [
            ".loc[lambda df: $0, :]"
        ],
        "description": ".loc[lambda df: df['a'] > 0, :]"
    },
    ".assign(**{'new_col': 123})": {
        "prefix": ".as",
        "body": [
            ".assign(**{'$1': $0})"
        ],
        "description": ".assign(**{'new_col': 123})"
    },
    "lambda df: ": {
        "prefix": "ldf",
        "body": [
            "lambda df: "
        ],
        "description": "lambda df: "
    },
    ".sort_values(by=['some_col'], ascending=[True]}])": {
        "prefix": ".sv",
        "body": [
          ".sort_values(by=['${1}'], ascending=[${2:True}])"
        ],
        "description": ".sort_values(by=['some_col'], ascending=[True]}])"
    },
    ".drop(columns=['some_col'])": {
        "prefix": ".dc",
        "body": [
            ".drop(columns=['$0'])"
        ],
        "description": ".drop(columns=['some_col'])"
    },
    ".drop_duplicates(['some_col'], keep='first')": {
        "prefix": ".dd",
        "body": [
          ".drop_duplicates(['${1}'], keep=${2:'first'})"
        ],
        "description": ".drop_duplicates(['some_col'], keep='first')"
    },
    ".apply(some_func, axis=1)": {
        "prefix": ".ap1",
        "body": [
            ".apply($0, axis=1)"
        ],
        "description": ".apply(some_func, axis=1)"
    },
    ".rename(columns={'old_name': 'new_name'})": {
        "prefix": ".rn",
        "body": [
            ".rename(columns={'$1': '$0'})"
        ],
        "description": ".rename(columns={'old_name': 'new_name'})"
    }
}
```