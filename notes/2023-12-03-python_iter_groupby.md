---
date: 2023-12-03T17:17:00+08:00
title: 记 Python itertools.groupby 数据为空的坑
category: 想法与笔记

tags: 
 - Python

toc: false
toc_fold: false

updated: 
copyright: true  # 暂不支持
---

在进行数据清洗时候，遇到这样一个需求，抽象后如下：

原数据为一个文本文件，文本按照问题和答案分行交替出现，一个问题后面可能会跟着多行答案，例如：

```python
x = """
Q:1+1=?
A:2

Q:水果有哪些
A:苹果
A:香蕉
A:桔子
"""
```

<!--more-->

我们需要按照问题答案配对清洗出来，多行答案情况需要合并成一个答案。想要的清洗结果为：

```python
[
    {
        "question": "Q:1+1=?",
        "answer": "A:2"
    },
    {
        "question": "Q:水果有哪些",
        "answer": "A:苹果,A:香蕉,A:桔子"
    }
]
```

我最初的代码实现如下：

```python
import itertools

x = x.strip()
groups = list(itertools.groupby(x.split('\n'), lambda x: x.startswith('Q')))

result = []
for i in range(0, len(groups), 2):
    (_, question_iter), (_, answer_iter) = groups[i: i + 2]
    result.append({
        "question": ",".join(question_iter),
        "answer": ",".join(answer_iter)
    })

```

这里我使用 `itertools.groupby` 方法把以Q开头的行或连续的以A开头的行分别聚合起来，groupby方法以一个迭代器形式返回这些聚在一起的组。接着，我将这个迭代器转换成一个list，每2个组（一个Q开头的组与一个A开头的组）一起处理，转换成最后的格式。

但上述代码运行结果是错误的。把list里的迭代器join后的字符串为空。

```python
print(result)

# [
#     {
#         "question": "",
#         "answer": ""
#     },
#     {
#         "question": "",
#         "answer": ""
#     }
# ]
```

在网络上查找相关资料后，了解到这个坑出现的原因与`itertools.groupby`方法返回迭代器背后的原理有关系：首先，groupby返回结果是一个迭代器，每一个迭代器的元素是group的key和该group下所有元素（也以迭代器形式返回）。我们姑且将外层不同 group 之间遍历的迭代器称为大迭代器，里层同一group的元素遍历的迭代器称为小迭代器。

整个大小迭代器的遍历其实都是基于原始输入的迭代器的遍历：小迭代器首先会遍历原始迭代器，直到遇到不同key的元素，这时大迭代器结束生成当前group的元素，并且开始生成下一个group元素。由此，当遍历大迭代器时，之前已经遍历完成的 group 对应小迭代器也遍历完成，即到达迭代器末尾状态，那么相应join后的字符串结果就为空了。

为了得到期望的结果，需要在遍历大遍历器时，将每个小遍历器内元素先存在list中。后面进行`for i in range(0, len(groups), 2):`遍历时，再对list中小遍历器的文本元素join时就得到正常结果了。修改后的代码如下：

```python
import itertools
x = x.strip()
groups = list(_, list(group) for  _, group in itertools.groupby(x.split('\n'), lambda x: x.startswith('Q')))

result = []
for i in range(0, len(groups), 2):
    (_, question_lst), (_, answer_lst) = groups[i: i + 2]
    result.append({
        "question": ",".join(question_iter),
        "answer": ",".join(answer_iter)
    })


print(result)

# [
#     {
#         "question": "Q:1+1=?",
#         "answer": "A:2"
#     },
#     {
#         "question": "Q:水果有哪些",
#         "answer": "A:苹果,A:香蕉,A:桔子"
#     }
# ]
```





