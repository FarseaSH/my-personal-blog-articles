---
title: 使用AutoHotkey置顶Windows系统的窗口
date: 2024-06-06T19:14:00+08:00
updated:
copyright: true  # 暂不支持
category: 想法与笔记
tags:
    - 工具
    - 效率
toc: false
toc_fold: false
draft: false
---

公司的电脑是Windows的，在平常办公过程中，有时会遇到需要将某个窗口置顶，显示在屏幕最前方，打开其它窗口时不会被遮挡住。例如，需要将记录了提醒信息的便签置顶，需要将参考的图片/网页置顶等。自己查了相关资料，发现可以使用AutoHotkey实现任意窗口的置顶，这里简单记录一下脚本配置。

<!--more-->

具体的AutoHotkey实现代码如下：

```
<#Esc::
WinSet, AlwaysOnTop, -1, A
Return
```

通过这段配置，想要置顶某个窗口，只需先点击激活该窗口，然后按下`左边Windows键 + Esc`，就可以置顶该窗口，要想取消置顶，只需重新点击该窗口并再次按下`左边Windows键  + Esc` 即可。

另外，我是用的 AutoHotkey 版本为 v1.1.34.04，如果是其它版本，可能需要确定置顶函数的名称和参数。
