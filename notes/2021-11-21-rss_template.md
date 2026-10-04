---
date: 2021-11-21T00:44:00+08:00
title: RSS Feed xml格式模版与标签归纳
category: 想法与笔记

tags: 
 - RSS

toc: true
toc_fold: false

updated: 
copyright: true  # 暂不支持
---

最近在做的几个个人项目，都会遇到RSS feed的开发，这里对rss开发相关的资料做一个汇总，方便以后开发使用。
<!--more-->

## 模版

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">

<channel>
  <title>频道的名称</title>
  <link>https://www.w3schools.com</link>
  <description><![CDATA[频道的描述]]></description>
    
  <item>
    <title>第一篇文章的标题</title>
    <link>https://www.w3schools.com/xml/xml_rss.asp</link>
    <description><![CDATA[第一篇文章的内容]]></description>
  </item>
    
  <item>
    <title>第二篇文章的标题</title>
    <link>https://www.w3schools.com/xml</link>
    <description><![CDATA[第二篇文章的内容]]></description>
  </item>
    
</channel>

</rss>
```

这里主要借鉴了[XML RSS - W3School](https://www.w3schools.com/xml/xml_rss.asp)。

## 标签（字段）含义

### `<channel>`下标签

| 标签        | 选填与否 | 标签说明          | 例子                                                         |
| ----------- | -------- | :---------------- | :----------------------------------------------------------- |
| title       | **必填** | RSS源的名称       | GoUpstate.com News Headlines                                 |
| link        | **必填** | RSS信息源所在网页 | http://www.goupstate.com/                                    |
| description | **必填** | 这个RSS源的介绍   | The latest news from GoUpstate.com, a Spartanburg Herald-Journal Web site. |
| language       | 选填 | 该RSS源使用的语言，可以参考 [HTML 语言代码参考手册(W3School)](https://www.w3school.com.cn/tags/html_ref_language_codes.asp)查找具体的语言代码 | zh                                                           |
| copyright      | 选填 | 文章的版权说明                                             | Copyright 2002, Spartanburg Herald-Journal                   |
| managingEditor | 选填 | 该RSS源内容方面负责人的Email地址                           | geo@herald.com (George Matesky)                              |
| webMaster      | 选填 | 该RSS源技术方面负责人的Email地址                           | betty@herald.com (Betty Guernsey)                            |
| pubDate        | 选填 | 该RSS源发布的时间，需要符合 [RFC 822](http://asg.web.cmu.edu/rfc/rfc822.html) 格式 | Sat, 07 Sep 2002 0:00:01 GMT                                 |
| lastBuildDate  | 选填 | 该RSS源最后更新的时间                                        | Sat, 07 Sep 2002 9:42:31 GMT                                 |
| category       | 选填 | 该RSS源所属的分类（可添加多项） | <category>Newspapers</category>                              |
| generator      | 选填 | 生成该RSS的程序                                              | MightyInHouse Content System v2.3                            |
| docs           | 选填 | 该RSS源格式说明的网页URL地址 | http://backend.userland.com/rss                              |
| cloud          | 选填 | Allows processes to register with a cloud to be notified of updates to the channel, implementing a lightweight publish-subscribe protocol for RSS feeds. More info [here](https://validator.w3.org/feed/docs/rss2.html#ltcloudgtSubelementOfLtchannelgt). | <cloud domain="rpc.sys.com" port="80" path="/RPC2" registerProcedure="pingMe" protocol="soap"/> |
| ttl            | 选填 | ttl是time to live的缩写，用于说明该RSS保持即时性的时间，即多长时间内（用分钟数表示）无需再从该RSS源拉取新的更新。 | <ttl>60</ttl>                                                |
| image          | 选填 | 用于的展示该RSS源的GIF, JPEG 以及 PNG 地址 |                                                              |
| textInput      | 选填 | Specifies a text input box that can be displayed with the channel. More info [here](https://validator.w3.org/feed/docs/rss2.html#lttextinputgtSubelementOfLtchannelgt). |                                                              |
| skipHours      | 选填 | A hint for aggregators telling them which hours they can skip. More info [here](http://backend.userland.com/skipHoursDays#skiphours). |                                                              |
| skipDays       | 选填 | A hint for aggregators telling them which days they can skip. More info [here](http://backend.userland.com/skipHoursDays#skipdays). |                                                              |

### `<Item>`下标签

| 标签        | 选填与否 | 标签说明                         | 例子                                                         |
| ----------- | -------- | -------------------------------- | ------------------------------------------------------------ |
| title       | **必填** | 文章的标题                       | Venice Film Festival Tries to Quit Sinking                   |
| link        | **必填** | 文章的原文链接                   | http://www.nytimes.com/2002/09/07/movies/07FEST.html         |
| description | **必填** | 文章内容                         | Some of the most heated chatter at the Venice Film Festival this week was about the way that the arrival of the stars at the Palazzo del Cinema was being staged. |
| guid        | 选填     | 文章的id                         | <guid isPermaLink="true">http://inessential.com/2002/09/01.php#a2</guid> |
| author      | 选填     | 文章作者的Email地址              | oprah@oxygen.net                                             |
| pubDate     | 选填     | 文章发布的时间                   | Sun, 19 May 2002 15:21:36 GMT                                |
| category    | 选填     | 该篇文章所属的分类（可添加多项） | Simpsons Characters                                          |
| source      | 选填     | 该篇文章的RSS源                  | <category domain="http://www.fool.com/cusips">MSFT</category> |
| comments    | 选填     | 该篇文章评论区的URL网址          | http://www.myblog.org/cgi-local/mt/mt-comments.cgi?entry_id=290 |
| enclosure   | 选填     | 该篇文章附加的媒体资源           | <enclosure url="http://live.curry.com/mp3/celebritySCms.mp3" length="1069871" type="audio/mpeg"/> |

## CDATA

对于`<description>`标签里面的内容，我们最好加上CDATA标识，以避免产生解析上的歧义与错误。

关于<![CDATA[]]>的作用，在Stack Overflow上有这样的解释

> A CDATA section is "a section of element content that is marked for the parser to interpret as only character data, not markup."
>
> -- [What does < ![CDATA[]] > in XML mean?](https://stackoverflow.com/questions/2784183/what-does-cdata-in-xml-mean)

翻译过来就是，被标记为CDATA的section，会被解析器解析为纯字符串文本，而非xml代码块。

