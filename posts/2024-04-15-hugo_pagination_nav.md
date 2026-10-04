---
title: Hugo主题开发系列丨添加分页导航栏
date: 2024-04-15T20:56:00+08:00
updated:
copyright: true  # 暂不支持
category: 经验
tags:
    - Hugo
toc: true
toc_fold: false
draft: false
---

在进行Hugo主题开发时，首页部分往往会用来呈现博客文章的摘要，为了避免文章太多而造成首页显示内容过长的问题，我们会将文章分页显示，分页显示离不开分页导航栏的设计与开发。前不久自己在开发Moment主题的时候，开发了一个简单的分页导航栏，这里做一个简单的记录，方便后来的主题开发者参考。

<!--more-->

开发栏模块的设计上，主要达到如下几个需求：

1. 可以点击切换到上一页和下一页，并且在首页和尾页时，上一页或下一页的按钮会灰掉
2. 可以点击快速进入到首页或尾页
3. 在页数很大时，分页导航栏会随着所在页数正确进行中间段页码的省略，以避免分页导航栏占用空间过长
4. 鼠标指针放置在每个按钮时，会有样式变化的响应

最后开发的效果如下

![分页导航栏效果图](https://s21.ax1x.com/2024/07/17/pkoeALd.png)

## 代码部分

首先，如果要开启Hugo分页功能，可以参考Hugo官方文档分页部分的设置（[链接](https://gohugo.io/templates/pagination/)。主题开发时，Hugo的分页会用到 paginator 变量的调用，详细调用方法可以参考这篇官方文档（[链接](https://gohugo.io/methods/page/paginator/)）。

以下是所有相关代码

Html 部分，注意将 `$paginator :=` 后面部分替换相应 paginator 即可，

```html
{{/* 下面部分替换成你的paginator */}}
{{ $paginator := .paginator }}
<div class="pagination">
    
    {{ if $paginator.HasPrev }}
    <a class="page-link" href="{{ $paginator.Prev.URL }}">
        <div class="page-item">< Prev</div>
    </a>
    {{ else }}
        <div class="page-item disabled">< Prev</div>
    {{ end }}


    {{/* If the number of pagers is more than 5, use `...` to skip some pages */}}
    {{ if (le $paginator.TotalPages 5) }}
        {{ $current_num := $paginator.PageNumber }}
        {{ range $i, $pager := $paginator.Pagers }}
            {{ if (eq $current_num $pager.PageNumber) }}
            <div class="page-item active">{{ $pager.PageNumber }}</div>
            {{ else }}
            <a class="page-link" href="{{ $pager.URL }}"><div class="page-item">{{ $pager.PageNumber }}</div></a>
            {{ end }}
        {{ end }}

    {{ else }}
        {{ $first_page_num := 1 }}
        {{ $second_page_num := 2 }}
        {{ $last_page_num := $paginator.TotalPages }}
        {{ $second_last_page_num := (add $paginator.TotalPages -1) }}
        {{ $third_last_page_num := (add $paginator.TotalPages -2) }}
        {{ $current_num := $paginator.PageNumber }}

        {{/* place 1 */}}
        {{ if (eq $current_num $first_page_num) }}
            <div class="page-item active">{{ $current_num }}</div>
        {{ else }}
            <a class="page-link" href="{{ $paginator.First.URL }}"><div class="page-item">1</div></a>
        {{ end }}

        {{/* place 2 */}}
        {{ if (eq $current_num $second_page_num) }}
            <div class="page-item active">{{ $current_num }}</div>
        {{ else if (le $current_num 3)}}
            <a class="page-link" href="{{ $paginator.First.Next.URL }}"><div class="page-item">{{ $second_page_num }}</div></a>
        {{ else }}
            <div class="page-item">...</div>
        {{ end }}

        {{/* place 3 */}}
        {{ if (le $current_num $second_page_num)}}
            <a class="page-link" href="{{ $paginator.First.Next.Next.URL }}"><div class="page-item">3</div></a>
        {{ else if (ge $current_num $second_last_page_num) }}
            <a class="page-link" href="{{ $paginator.Last.Prev.Prev.URL }}"><div class="page-item">{{ $third_last_page_num }}</div></a>
        {{ else }}
            <div class="page-item active">{{ $current_num }}</div>
        {{ end }}

        {{/* place 4 */}}
        {{ if (eq $current_num $second_last_page_num) }}
            <div class="page-item active">{{ $current_num }}</div>
        {{ else if (ge $current_num $third_last_page_num) }}
            <a class="page-link" href="{{ $paginator.Last.Prev.URL }}"><div class="page-item">{{ $second_last_page_num }}</div></a>
        {{ else }}
            <div class="page-item">...</div>
        {{ end }}

        {{/* place 5 */}}
        {{ if (eq $current_num $last_page_num) }}
            <div class="page-item active">{{ $current_num }}</div>
        {{ else }}
            <a class="page-link" href="{{ $paginator.Last.URL }}"><div class="page-item">{{ $last_page_num }}</div></a>
        {{ end }}
    {{ end }}


    {{ if $paginator.HasNext }}
        <a class="page-link" href="{{ $paginator.Next.URL }}">
            <div class="page-item">Next ></div>
        </a>
    {{ else }}
        <div class="page-item disabled">Next ></div>
    {{ end }}

</div>
```

css 样式文件，可以根据自己的主题，进行样式的微调

```css
.pagination {
	 display: flex;
	 margin-top: 20px;
	 margin-bottom: 30px;
}
 .pagination .page-item {
	 padding-left: 15px;
	 padding-right: 15px;
	 padding-top: 10px;
	 padding-bottom: 10px;
}
 .pagination .page-item.active {
	 background-color: #3a6ba5;
	 color: #fff;
}
 .pagination .page-item.disabled {
	 color: #919eb1;
}
 .pagination a {
	 color: #21365c;
	 text-decoration: none;
}
 .pagination .page-link .page-item:hover {
	 background-color: #f5f5f5;
}
```