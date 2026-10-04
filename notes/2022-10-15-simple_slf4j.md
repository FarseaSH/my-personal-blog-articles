---
date: 2022-10-15T19:03:00+08:00
title: slf4j 本地日志输出
category: 想法与笔记
tags: 
 - Java
toc: false
toc_fold: false
updated: 
copyright: true  # 暂不支持
draft: false
---
在进行Java开发时，我们常需要打印日志来帮助我们检查运行结果，定位问题。slf4j是常用的日志框架，但其只是日志框架的一部分（即所谓的设计模式中的门面），真正日志记录的实现还需要相应的后端支持(如在生产环境配置的log4j2)，这意味着，在本地机器中如果没有相关配置，代码中slf4j记录的日志可能无法直接查看。我们可以通过简单的配置使我们在本地启动服务时候也能顺利打印出日志。

<!--more-->

这里我们使用 slf4j-simple 包，它可以将日志内容通过标准输出的方式打印出来，具体操作如下：

1. 首先将其jar包下载，下载地址 [https://repo1.maven.org/maven2/org/slf4j/slf4j-simple/2.0.7/slf4j-simple-2.0.7.jar](https://repo1.maven.org/maven2/org/slf4j/slf4j-simple/2.0.7/slf4j-simple-2.0.7.jar)

![IntelliJ 中导入 slf4j-simple 包](https://s1.ax1x.com/2023/07/29/pPS7xWq.png)

2. 将配置到运行环境中，以IntelliJ为例，打开`project structure`，选择`Platform Setting`下的`SDKs → Classpath`点击加号将刚才下载的jar包导入。

这样在运行时，我们在IntelliJ 运行时，可以在运行窗口看到日志输出。

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Slf4JSimpleDemo {
    public static void main(String[] args) {
        log.info("Hello World");
    }
}
```

![通过标准输出打印的日志内容](https://s1.ax1x.com/2023/07/29/pPSHFw4.png)

