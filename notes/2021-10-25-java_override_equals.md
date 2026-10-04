---
date: 2021-10-25 11:52:24
title: Java重写equals方法的一些批注
category: 想法与笔记
updated:
copyright: true  # 暂不支持
tags:
    - Java
toc: true
toc_fold: false
draft: false
---

最近在Java开发过程中，遇到了一个过滤的需求，需要从`List`函数中，剔除给定的元素，这些元素是自定义类。一个比较简洁的写法是用`remove()`函数，这里面涉及到重写类的比较方法`equals()`。这里简单的记录一下一些要点。

<!--more-->

## 重写`equals()`的代码片段

```java
@Override
public boolean equals(Object obj) {
    if (obj == null) {
        return false;
    }

    if (obj.getClass() != this.getClass()) {
        return false;
    }

    final Person other = (Person) obj;

    // START 修改以下部分
    if ((this.name == null) ? (other.name != null) : !this.name.equals(other.name)) {
        return false;
    }

    if (this.age != other.age) {
        return false;
    }
    // END 修改

    return true;
}

@Override
public int hashCode() {
    int hash = 3;
    hash = 53 * hash + (this.name != null ? this.name.hashCode() : 0);
    hash = 53 * hash + this.age;
    return hash;
}

public int getAge() {
    return age;
}

public void setAge(int age) {
    this.age = age;
}

public String getName() {
    return name;
}

public void setName(String name) {
    this.name = name;
}
```

## 理论点

为了使代码更为完整健壮，重写`equals()`的同时最好也重写`hashCode()`。

关于重写`equals()`与`hashCode()`的要求，我在Stack Overflow找到一篇不错的回答。

> equals() must define an equivalence relation (it must be reflexive, symmetric, and transitive). In addition, it must be consistent (if the objects are not modified, then it must keep returning the same value). Furthermore, o.equals(null) must always return false.
> 
> hashCode() must also be consistent (if the object is not modified in terms of equals(), it must keep returning the same value).
> 
> The relation between the two methods is:
>
> Whenever a.equals(b), then a.hashCode() must be same as b.hashCode().

翻译一下就是：

> equals() 必须定义一个等价关系（它必须满足自反性、对称性和传递性）。此外，它必须是一致的（如果对象没有被修改，那么它必须一直返回相同的值）。此外，o.equals(null)必须总返回false。
>
> hashCode() 也必须是一致的（如果没有修改对象的equals()方法，它必须一直返回相同的值）。
>
> 这两个方法之间的关系是：
>
> 若a.equals(b)为true，则一定能推出a.hashCode()与b.hashCode()相同。