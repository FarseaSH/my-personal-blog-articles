---
date: 2021-10-26 12:10:00
title: Java自定义类比较的代码片段
category: 想法与笔记
updated: 
copyright: true  # 暂不支持
tags: 
    - Java
toc: false
toc_fold: false


draft: false
---

最近在Java开发过程中，需要取自定义数据类中域的最大/最小值，这里贴一下我的代码片段，方便以后遇到类似需求时候复用。

<!--more-->

主要用到`Collections.max()`方法和`Comparator`的重写。

关于`Comparator`顺便批注一点：`Comparator`的设计是`Comparator`类独立与需要比较的类之外的，不是在类里面写入类似的`Compare()`方法。

```java
int maxValue = Collections.max(listToBeCompared, new Comparator<YourDataType>() {
    @Override
    public int compare(YourDataType o1, YourDataType o2) {
    return o1.getSomeValue() < o2.getSomeValue() ? -1 : 1;
    }
}).getSomeValue();
```


