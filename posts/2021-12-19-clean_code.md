---
title: "一些好的代码习惯"
date: 2021-12-19T14:29:05+08:00
updated: 
copyright: true  # 暂不支持
category: 经验
tags:
    - 代码习惯
    - Python
toc: true
toc_fold: false
draft: false
---

对于代码，自己也算半个有代码洁癖的人。自己平常也会开发一些小玩意儿，写点个人项目，因为各种各样的事情耽误，可能一个个人项目的开发是断断续续的，中间有时候会间隔好几个月，这么长时间的间隔，之前代码的思路，细节处理早已忘记，这时候需要重读代码来熟悉项目。久而久之，随着项目代码的积累，代码的质量就尤为重要。

养成好的代码习惯，可以促使我们写出高质量，可读性更强的代码，以方便后期维护，提高工作效率。本文记录了我在开发过程中，认为较好的代码习惯，用以规范自己以后的开发习惯。我大致将好的习惯分为两类，一类是能优化代码视觉阅读习惯的，另一类是能降低代码逻辑思考负担的。这个习惯，有些来自于自己实际的经验，有些来自于我代码的阅读过程，还有些来自于相关资料。

<!--more-->

文章会不定期更新，扩充丰富我认为较好的代码习惯。

## 命名要有意义合理

变量/属性&方法/函数的名称应该是有意义，好的名称可以让人快速理解代码逻辑，坏的名称则可能让代码阅读者感到困惑，甚至会误导读者。

变量名称需要有意义，符合代码的逻辑，避免使用无意义的变量名称

```python
# Bad Practice
def getFinalScore(x):
    a = x[0]
    b = x[1]
    return a + b

# Good Practice
def getFinalScore(rawScoreList):
    chineseScore = rawScoreList[0]
		mathScore = rawScoreList[1]
    return chineseScore + mathScore
```

常见方法/函数命名：

- `getXXX`
- `setXXX`
- `findXXX`
- `calculateXXX`
- `toString`
- `string2List`

常见布尔类型(True/False)变量命名：

- `isPositive`
- `hasPositiveNum`
- `canXXX`

变量名可以附带上变量类型，加强可读性

- `xxxAsStr`
- `xxxList`
- `xxxMap`

函数名可以更加详细

```python
# Bad Practice
def findUserInfo(id):
    pass

# Good Practice
def findUserInfoById(id):
    pass
```

避免一些常见词的简写

```
Bad: cnt  --  Good: count
Bad: rst  --  Good: result
Bad: q    --  Good: query
```

class类名称，变量/属性名称用名词（布尔类型变量除外）

```python
# Bad Practice
class FastSort(Sort):
		...

# Good Practice
class FastSorter(Sorter):
		...
```

函数命名，也最好遵循一些习惯用法。

> 用 first、last 表示访问空间的包含范围； begin、end 表示访问空间的排除范围，即 end 不包含尾部。

## 函数/方法的调用，入参加上参数名称

函数/方法的调用，最好加上参数名称，以加强代码可读性。

```python
# Bad practice
y1 = ...
y2 = ...
sklearn.metrics.f1_score(y1, y2)

# Good practice
y1 = ...
y2 = ...
sklearn.metrics.f1_score(y_true=y1, y_false=y2)
```

## 数字加上意义

数值字面量如果单独出现，可以用变量替换，增强代码的可读性。

```python
# Bad practice
for i in range(50):
    # do something


# Good practice
MAX_ITER = 50
for i in range(MAX_ITER){
    # do something
}
```

## if 与 return/continue/break 控制语句的简写

在做if判断中，如果判断后执行语句带return/continue/break等其它流程控制语句，不必按照if + else的模式进行书写，只需开头加入if判断，return/continue/break会跳过后面的语句。这样可以达到精简代码、降低视觉信息量的目的。我们还可以更进一步，将if语句和后面的条件执行语句放到一行，来减少代码行数。

一个典型的例子是空指针的判断。

```python
# Bad practice
def hasElements(lst):
    if lst is None:
        return False
		else:
				return len(lst) > 0

      
# Good practice
def hasElements(x):
    if lst is None: return False
		return len(lst) > 0
```

## if condition return True 语句的简写

遇到 if condition return True/False，通常可以直接转换为 return condition，减少代码行数，降低视觉信息量。

```python
# Bad practice
def isPositive(x):
    if x > 0:
        return True
		else:
				return False

# Good practice
def isPositive(x):
    return x > 0
```

## 注释的使用

> “While comments are neither inherently good or bad, they are frequently used as a crutch. You should always write your code as if comments didn’t exist. This forces you to write your code in the simplest, plainest, most self-documenting way you can humanly come up with.” 
> 
> -- Jeff Atwood

注释可以帮助代码阅读者更好更快的理解代码。但是，不能过度依赖注释来增加代码可读性。好的代码应该尽可能不依赖注释，就能阐述清楚代码所代表的逻辑。关于注释的使用，我自己个人通常的习惯是：

1. 用注释去解释代码的大方向，而非具体实现的细节，除非写法逻辑不直观，并且直观的写法会造成性能牺牲。
2. 方法/函数应该加上注释。
3. 方法/函数的注释，最好加上用例。

## 拆分长句/拆分多层逻辑

如果单部分代码包含多个逻辑，可以将其拆分为多部分，以降低代码阅读者的思考负担。

```python
# Bad Practice
if datetime.now() > start_time and \\
    (user.permission == 'permanant' or \\
    (user.permission == 'temperory' and meeting.allow == 'all')):
    # do something

    
# Good Practice
is_started = datetime.now() > start_time

is_allowed_user_entrance_1 = (user.permission == 'permanant')
is_allowed_user_entrance_2 = (user.permission == 'temperory' and meeting.allow == 'all')
is_allowed_user_entrance = is_allowed_user_entrance_1 or is_allowed_user_entrance_2

if is_started and is_allowed_user_entrance:
    # do something
```

## 代码空行分块

用空行将代码分块，可以按照逻辑步骤分块，也可以按照语句相似性进行分块，以降低代码视觉负担。