---
date: 2021-10-27 16:27:00   
title: Java字符串转Map的代码片段
category: 想法与笔记

tags: 
 - Java


toc: false
toc_fold: false

updated: 
copyright: true  # 暂不支持
---

最近在进行Java开发过程中，完成需求中的一个步骤需要将字符串解析为Map格式，这里记录一下自己经过几次迭代后的代码片段，方便以后作为util类方法复用。

<!--more-->

方法的特点：

1. 分隔符的自定义：常用的`key1: value1; key2: value2`和`field1=value1, field2=value2`都能得到支持；
2. 被解析的字符串中多余的空格不会影响解析。


```java
public static Map<String, Double> str2Map(String str, String kvSp, String pairSp) {
    Map<String, Double> resultMap = new HashMap<String, Double>();

    if (fString != null and !StringUtils.isEmpty(str)) {
        for (String pair : str.split(pairSp)) {
            String[] kv = pair.split(kvSp);
            if (kv.length == 2) {
                retMap.put(kv[0].trim(), Double.parseDouble(kv[1].trim()));
            }
        }
    }

    return resultMap;
}
```

