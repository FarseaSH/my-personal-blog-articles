---
date: 2022-06-18T21:30:00+08:00
title: Hugo设置在新页面打开链接
category: 想法与笔记
tags: 
 - Hugo
toc: false
toc_fold: false
updated: 
copyright: true  # 暂不支持
draft: false
---

默认情况下，打开Hugo博客中的链接，会在当前浏览器页面上直接进行跳转的，而不会在新浏览器标签页中打开。这导致了较差的用户体验，并且还可能让读者跳出了网站，终止后续阅读，不利于网站更多内容的呈现。

我们可以进行如下简单的配置，实现在新标签页打开链接的优化：在`theme`目录下找到当前主题对应的文件夹目录，在其下`layouts\_default\_markup`路径（如果没有自行建立）添加文件`render-link.html`，将下面一段代码复制到文件里后，即可配置完成

<!--more-->

```
<a href="{{ .Destination | safeURL }}"{{ if strings.HasPrefix .Destination "http" }} target="_blank" rel="noopener noreferrer"{{ end }}>{{ .Text | safeHTML }}</a>
```

