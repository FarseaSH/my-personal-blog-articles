---
date: 2023-07-18T22:04:00+08:00
title: Pandas 列名变量生成辅助工具
category: 想法与笔记
tags: 
 - Python
 - Pandas
toc: false
toc_fold: false
updated: 
copyright: true  # 暂不支持
draft: false
---

在用Python Pandas处理数据时候，我们经常会去做选取某一列的操作(e.g. `df['some_col']`)。这段代码需要敲击引号然后输入字符串，如果列名为中文，还需要切换输入法，非常影响开发的流畅性。

一个解决办法是，将 DataFrame 中的每一列的列名赋值成一个变量，在需要输入列名时，只需要输入对应的变量即可，借助于编辑器的代码补全功能，可以让选取列代码的开发更为便捷、流畅。

<!--more-->

自己写了一个小工具，用于列名赋值到python变量的代码生成。只需将列名的list(可以用`df.columns.to_list()`生成)传入左边的输入框，点击转换即可生成相应的代码。如：

例子 `df` 是下面这一张表

```
>>> df

   姓名  体测成绩
0  张三    99
1  李四    80
2  王五    74
3  小红    95

>>> df.columns.to_list()

['姓名', '体测成绩']
```

用该工具生成的辅助代码如下：

```python
xingming = '姓名'
tice_chengji = '体测成绩'
```

这个工具使用Python写的，利用jieba进行中文分词，再用`pypinyin` 包进行中文到英文拼音的转换，最后用pywebio完成前端UI的展示。

这个小工具的代码如下：

```python
from pypinyin import lazy_pinyin
from pywebio.pin import *
from pywebio.output import *
from pywebio import *
import pyclip
import jieba

def get_chinese_pinyin_field(text):
    return "_".join("".join(lazy_pinyin(cut_word)) for cut_word in jieba.cut(text))

def transform(input_text):
    input_list = eval(input_text)
    return "\n".join(f"{get_chinese_pinyin_field(text)} = '{text}'" for text in input_list)

def copy():
    pyclip.copy(pin.pin['output'])
    put_text("copied")

def main():
    put_textarea("input", label="input（放一个string list）")
    put_button("转换", onclick=lambda: pin_update("output", value=transform(pin.pin["input"])), color="light")
    put_textarea("output", label="output", rows=15)
    put_button("copy", onclick=copy, color="light")

if __name__ == '__main__':
    start_server(main, port=1234)
```
