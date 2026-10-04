---
title: Python Notion API 开发总结
date: 2024-08-07T22:11:00+08:00
updated:
copyright: true  # 暂不支持
category: 经验
tags:
    - Python
    - None
toc: true
toc_fold: false
draft: false
---

[Notion](https://www.notion.so/) 是当下一款火热的 All in One 软件，其多样且高度自定义的 block，与主打的完整全面的 database 功能，满足各种用户各类使用场景的需要，文档，知识维基、个人笔记、个人任务列表等都可以很好得到支持。除了本身优秀的产品逻辑，Notion 还提供了api接口，使得用户通过代码完成相关内容操作，如 database 的增删查改等。我自己也开发了许多脚本，调用这些 api，作为自己个人工作流的一部分，如检查 todo list 中的任务每天是否按时完成，查询财务清单中每月每个类别的花费开支是多少等。这些脚本是用 Python 开发，主要用到了这个[notion-client](https://github.com/ramnes/notion-sdk-py)库。本文总结了这些 Notion 脚本开发过程中常用的代码片段与自己封装的一些 helper 函数，方便后来有需要进行 Notion api 调用开发的开发者参考。

<!--more-->

## 基本操作

### client的生成

所有操作都是基于`Client`的方法，所以我首先需要声明`Client`，代码如下。其中参数 auth 为Notion Integration 的 secret。这个 secrete 可以从 Notion [My Integrations](https://www.notion.so/my-integrations) 页面中，在所要用的 automation 中找到，形如 `secret_Sjrb9wHo2MS6EB3gjpMbRb6g7h35qIx8uVnve7izuIdV`

```python
from notion_client import Client

client = Client(auth="")
```

### 检索 database 符合条件的页面

获取 database 中满足检索条件的 page。检索条件filter字段的写法（不同类型字段有不同的可检索条件与写法）可以参考官方文档[https://developers.notion.com/reference/post-database-query-filter](https://developers.notion.com/reference/post-database-query-filter)

```python
filter_example = {
    "property": "状态",
    "select": {
        "equals": "正在进行"
    }
}

client.databases.query(notion_database_id, filter=filter_example)
```

注意

1. 这个函数只会返回 database 中 page 的 id 与各个字段的值，不会返回 page 页面内block的内容，若需要返回某个 page 里面的 block，可以参考[获取某个 page 下的 block](#%E8%8E%B7%E5%8F%96%E6%9F%90%E4%B8%AA-page-%E4%B8%8B%E7%9A%84-block)章节
2. 返回的page数量是有一定上限的，不一定会返回所有结果，如果返回page的数量超过了上限，需要分批请求返回结果，我后面封装了一个函数，实现了这个分批请求的逻辑，函数直接返回所有结果，参见 [获取所有-database-中符合条件的页面](#获取所有-database-中符合条件的页面) 章节

### 更新 database 某个页面的属性

更改 database 中某个页面的属性，也即更改 database 中某行的字段值，可以参考如下代码。各种不同类型字段（如文本、数字、单选、多选等）更新值的格式，即下面python properties 参数写法，可以参考 https://developers.notion.com/reference/page-property-values

```python
properties = {
    "checkbox_field_name": {
        "checkbox": True
    },
    "number_field_name": {
        "number": 1.0
    }
}

client.pages.update(block_id, properties=properties)
```

### 获取某个 page 下的 block

获取某个 page 下的 block，也即获取某个页面下的内容，可以参照以下代码

```python
client.blocks.children.list(notion_page_id)
```

需要注意几点

1. 这个函数只会返回页面下的一级block，不会返回一级block下的子block，例如，用tab缩进后的段落不会被返回，分栏（i.e 分 cloumn）下的block也不会被返回。如果要返回这些子block需要再次调用下面这个 `client.blocks.children.list` 函数，并传如父block的id
2. 返回的block数量是有一定上限的，不一定会返回所有结果，如果返回page的数量超过了上限，需要分批请求返回结果，我后面封装了一个函数，实现了这个分批请求的逻辑，函数直接返回page下所有block（但不包括block下的子block），参见[获取某个page下的所有 block](#获取某个-page-下的所有-block不包括block下的子block)

### 向某个 page 末尾添加 block

往某个page添加block，或者往某个block下添加子block，可以参考下面代码。默认添加的位置是添加至页面或者子block的末尾。函数将block插入至某个特定的位置，只需在 `after` 参数下填入某个block id，新block即会插入在这个block之后。

```python
blocks_to_append = [{
    "type": "paragraph",
    "paragraph": {
        "rich_text": [{
            "type": "text",
            "text": {
                "content": "Hello World!",
                "link": None
            }
        }],
        "color": "default"
    }
}]

client.blocks.children.append(page_id, children=blocks_to_append)
```

## 封装函数

### 获取所有 database 中符合条件的页面

原生的`client.databases.query`函数在满足filter条件的page数量较多时，无法一次性获取所有的page。我简单开发了一个helper函数，对上面函数封装，通过多次调用该函数实现所有page的获取。

```python
def fetch_all_notion_database_pages(notion_database_id: str，filter_: dict) -> 'list[dict]':
    has_more, start_cursor = True, None
    result = []
    while has_more:
        resp: dict = client.databases.query(notion_database_id, filter=filter_, start_cursor=start_cursor)
        result.extend(resp['results'])
        has_more = resp['has_more']
        start_cursor = resp['next_cursor']
    
    return result
```

### 获取某个 page 下的所有 block（不包括block下的子block）

类似的，原生的`client.blocks.children.list`函数在block数量较多时，无法一次性获取所有的block。我也简单封装了一下这个函数，通过多次调用该函数实现所有block的获取。具体代码如下：

```python
def fetch_all_notion_children_blocks(notion_page_id: str) -> 'list[dict]':
    has_more, start_cursor = True, None
    result = []
    while has_more:
        resp: dict = client.blocks.children.list(notion_page_id, start_cursor=start_cursor)
        result.extend(resp['results'])
        has_more = resp['has_more']
        start_cursor = resp['next_cursor']
    
    return result
```

### Notion 富文本转换为 Markdown文本或普通文本

在进行notion api开发时候，经常会遇到block或property中文本的提取。notion api返回结果中，这些文本是在rich_text这个字段以jsonObject形式返回，里面除了文本外还包括字体、颜色等信息。这里我开发了一个清洗函数，可以将该jsonObject转化为纯文本或markdown文本。

```python
def extract_rich_text(rich_text: 'list[dict]', only_plain_text=False) -> str:
    result: 'list[str]' = []
    for rich_text_part in rich_text:
        text = rich_text_part['plain_text']

        if only_plain_text:
            result.append(text)
            continue

        if rich_text_part['annotations']['bold']: text = f"**{text}**"
        if rich_text_part['annotations']['italic']: text = f"*{text}*"
        if rich_text_part['annotations']['strikethrough']: text = f"~~{text}~~"
        if rich_text_part['annotations']['code']: text = f"`{text}`"
        if 'href' in rich_text_part and rich_text_part['href']:
            text = f"[{text}]({rich_text_part['href']})"

        result.append(text)
    
    return "".join(result)
```

### 获取某个页面下包括子block在内的所有block

原生notion api获取页面下block的函数 `client.blocks.children.list` 是无法直接获取block下的子block的。我在开发[Notion2Moment](https://github.com/FarseaSH/Notion2Moments)项目时，需要对整个notion页面做解析，这要求获取页面下所有block及其全部层级子block，并且还需要保留同层级前后顺序与不同层级父子关系。为了这个需求自己写了一套代码，整个代码实现两个类 `NotionBlockTree` 和 `NotionBlockTreeFetch`，其中 `NotionBlockTree` 是一个抽象的notion block节点，`child_block`属性指向其第一个子block，`next_block`属性指向平级后面一个block。`val`属性是该notion block的实际内容。

NotionBlockTreeFetch 类实现了获取notion page下所有block和子block的逻辑，最后返回一个 NotionBlockTree 对象，这个NotionBlockTree对象代表请求的page，其child_block指向实质的page内容。后续对page内容操作的逻辑在这个对象之上实现即可。

```python
from typing import List
from typing_extensions import Union

class NotionBlockTree:
    def __init__(self, notion_id: str, val: Union[dict, None]) -> None:
        self.notion_id: str = notion_id
        self.val: Union[dict, None] = val
        self.child_block: Union[NotionBlockTree, None] = None
        self.next_block: Union[NotionBlockTree, None] = None

class NotionBlockTreeFetcher:
    """
    用于给定某个 notion page 的 id，生成对应的 notion block树结构
    """
    def __init__(self, root_notion_id: str) -> None:
        self.root_block: NotionBlockTree = NotionBlockTree(root_notion_id, None)
        self.blocks_to_query_children: List[NotionBlockTree] = [self.root_block]
        self.is_fetched = False

    def fetch(self):
        while self.blocks_to_query_children:
            current_block_to_fetch_children = self.blocks_to_query_children.pop(0)

            children_blocks = fetch_all_notion_children_blocks(current_block_to_fetch_children.notion_id)
            blk = dummy = NotionBlockTree("", None)
            for child_block in children_blocks:
                _next_block = NotionBlockTree(notion_id=child_block['id'], val=child_block)

                if child_block['has_children']:
                    self.blocks_to_query_children.append(_next_block)
                
                blk.next_block = _next_block
                blk = blk.next_block

            current_block_to_fetch_children.child_block = dummy.next_block
        
        self.is_fetched = True

    def get_result(self) -> NotionBlockTree:
        if not self.is_fetched: raise AssertionError("`fetch` method has not run.")
        return self.root_block
```

请求的代码可以参考如下：

```python
page_id = "YOUR_PAGE_ID"
nbtf = NotionBlockTreeFetcher(page_id)
nbtf.fetch()
block_tree = nbtf.get_result()
```