---
title: Python 实现 Java Stream 接口
date: 2024-07-15T21:20:00+08:00
updated:
copyright: true  # 暂不支持
category: 经验
tags:
    - Python
    - Java
toc: true
toc_fold: false
draft: false
---

之前开发过一些搜索策略的Java代码，很大部分使用了Stream API对列表中元素进行过滤、排序等操作。后面切换到Python做一些数据分析工作时，有时也会用到类似的逻辑，但是Python并没有对应的Stream API，习惯了这套写法后，再用一些原生基础的列表函数操作会稍微有一些不习惯。后面抽空找时间简单写了一套 Python Stream API，方便有对应需求时候使用。这篇文章对此做一个简单的介绍，并附上相关代码。

<!--more-->

## 说明

代码主要实现了 Stream 类的所有 api，调用排序函数时常用到的 Comparator 类，与用于输出结果的 Collector 类。函数名称与输入类型与Java相对应的包都保持一致（除了驼峰命名改成Python常用下划线分隔命名），使用过Java相应包的开发者可以没有学习难度的使用这套api。此外，所有方法都加上了类型注释，配合常用的python开发环境，可以最大程度避免开发阶段的代码错误。

代码实现上，Stream类尽可能利用Python迭代器，来减少计算过程内存的占用与多余变量的生成。Stream中的方法基本上使用相关Python原生函数（如排序时使用sorted,，过滤使用 filter 函数，任一匹配函数使用 any函数），来保证实现的效率。

最后，在给出实现代码前，我们先看一个实际开发过程中所遇到的例子，来感受使用这套 Python Stream api 与常规写法的风格的差异。

考虑新闻推荐列表生成的场景，有这样一个业务逻辑要实现，得到召回的推荐新闻列表后：

1. 首先，只保留最近3天的发布的新闻
2. 其次，按照点击数分层从高到底排序，每1000为一层（即，1999点击量与1000点击量，但2000点击量高于两者），如果两条新闻点击数分层后的排名相同，则按照事先计算好的质量分排序
3. 接着，按照作者进行打散，即相同作者发布的第二条新闻应该置于其它作者发布的第一条新闻之后
4. 截取前30条新闻，并且只返回新闻id构成的列表

常规Python实现

```python
from pydantic import BaseModel
from datetime import date
from stream import *
from comparator import *

class News(BaseModel):
    id: int
    category: int
    author: str
    publish_date: date
    click_count: int
    quality_score: float


class MainRanker:
    @classmethod
    def rank(cls, news_list: list[News]) -> list[int]:
        # 1 只保留3天内的新闻
        news_within_time_range = []
        for news in news_list:
            if (date.today() - news.publish_date).days <= 3: news_within_time_range.append(news)
        
        # 2 按照 clickCount 分层排序（1000为一组），如果相同，进一步安排质量分从高到底排序
        news_within_time_range.sort(key=lambda x: (x.click_count // 1000, x.quality_score), reverse=True)
        
        # 3 作者打散逻辑
        author_set = set()
        first_part, second_part = [], []
        for news in news_within_time_range:
            if news.author not in author_set:
                first_part.append(news)
                author_set.add(news.author)
            else:
                second_part.append(news)
        
        news_shuffle_by_author = first_part + second_part
        
        # 4 截取前30
        news_with_limit = news_shuffle_by_author[:30]

        return [news.id for news in news_with_limit]
```

使用 Stream API 的写法

```python
from operator import attrgetter

class MainRankerWithStreamAPI:
    click_count_then_quality_comparator = (Comparator
        .comparing(lambda news: news.click_count // 1000, Comparator.reverse_order())
        .then_comparing(attrgetter('quality_score'), Comparator.reverse_order())
    )

    @classmethod
    def rank(cls, news_list: list[News]) -> list[int]:
        author_set = set()
        def author_set_add(author: str):
            result = author in author_set
            author_set.add(author)
            return result

        i = 0
        def get_inc_i():
            i += 1
            return i
        
        return (
            Stream.of(news_list)
                .filter(lambda news: (date.today() - news.publish_date).days <= 3)  # 1 只保留3天内的新闻
                .sorted(cls.click_count_then_quality_comparator)  # 2 按照 clickCount 分层排序（1000为一组），如果相同，进一步安排质量分从高到底排序
                .map(lambda news: (author_set_add(news.author), get_inc_i(), news))  # 3 作者打散逻辑
                .sorted(Comparator.natural_order())
                .map(itemgetter(2))
                .limit(30)  # 4 截取前30
                .map(attrgetter('id'))
                .collect(Collectors.to_list())
        )

```


相应在java中的实现，python实现与其几乎没有差异。

```java
public class MainRankerWithSreamAPI {
    private static final Comparator<News> clickCountThenQualityComparator = Comparator
            .<News, Integer>comparing(news -> news.getClickCount() / 1000, Comparator.reverseOrder())
            .thenComparing(News::getQualityScrore, Comparator.reverseOrder());

    public static List<Long> rank(List<News> newsList) {
        Set<String> authorSet = new HashSet<>();
        AtomicInteger i = new AtomicInteger(0);

        return newsList.stream()
                .filter(news -> Period.between(LocalDate.now(), news.getPublishDate().toLocalDate()).getDays() <= 3) // 1 只保留3天内的新闻
                .sorted(clickCountThenQualityComparator)  // 2 按照 clickCount 分层排序（1000为一组），如果相同，进一步安排质量分从高到底排序
                .map(news -> Triple.of(authorSet.add(news.getAuthor()) ? 0 : 1, i.getAndIncrement(), news))  // 3 作者打散逻辑
                .sorted()
                .map(Triple::getRight)
                .limit(30)       // 4 截取前30
                .map(News::getId)
                .collect(Collectors.toList());
    }
}
```



## 实现代码

```python
# stream.py

from typing import Callable, Collection, Optional, Iterable
from typing_extensions import Self

class Collector:
    def __init__(self, collection_func) -> None:
        if collection_func is not None:
            self.collect = collection_func
        
    def collect() -> Collection:
        raise NotImplementedError("`collect` method is not implemented")


class Collectors:
    PAIR_TO_MAP_COLLECTOR = Collector(dict)

    _LIST_COLLECTOR = Collector(list)
    _SET_COLLECTOR = Collector(set)
    

    @classmethod
    def to_list(cls) -> Collector:
        return cls._LIST_COLLECTOR


    @classmethod
    def to_set(cls) -> Collector:
        return cls._SET_COLLECTOR


    @classmethod
    def to_map(cls, key_mapper: Callable[[object], object], value_mapper: Callable[[object], object]) -> Collector:
        return Collector(lambda iter_: {key_mapper(x): value_mapper(x) for x in iter_})


class Stream:
    @staticmethod
    def of(iterable):
        return Stream(iterable)
    
    def __init__(self, iterable: Iterable) -> None:
        self._iterable: Iterable = iterable
    

    def map(self, mapper: Callable[[object], object]) -> Self:
        self._iterable = (mapper(e) for e in self._iterable)
        return self


    def flat_map(self, mapper: Callable[[object], Iterable]) -> Self:
        self._iterable = (flat_e for e in self._iterable for flat_e in mapper(e))
        return self


    def filter(self, predictate: Callable[[object], bool]) -> Self:
        self._iterable = filter(predictate, self._iterable)
        return self


    def sort(self, key=None, reverse=None) -> Self:
        params = {}
        if key is not None: params['key'] = key
        if reverse is not None: params['reverse'] = reverse

        self._iterable = sorted(self._iterable, **params)
        return self


    def collect(self, collector: Collector) -> Collection:
        return collector.collect(self._iterable)


    def any_match(self, predictate: Callable[[object], bool]) -> bool:
        return any(predictate(e) for e in self._iterable)


    def all_match(self, predictate: Callable[[object], bool]) -> bool:
        return all(predictate(e) for e in self._iterable)


    def for_each(self, action: Callable[[object], None]) -> None:
        for e in self._iterable: action(e)


    def find_first(self) -> Optional[object]:
        for e in self._iterable: return e


    def find_any(self) -> Optional[object]:
        return self.find_first()
```

```python
# comparator.py
from __future__ import annotations
from typing import Callable, Optional, Union


class Comparator:
    @classmethod
    def comparing(cls, key_extractor: Callable[[object], object], comparator: Comparator=None) -> Comparator:
        if comparator is not None:
            return KeyComparator(_key_extractor=key_extractor, _key_comparator=comparator)
        else:
            return KeyComparator(_key_extractor=key_extractor, _key_comparator=_natural_order_comparator)


    @classmethod
    def natural_order(cls):
        return _natural_order_comparator


    @classmethod
    def reverse_order(cls) -> Comparator:
        return ReversedOrderComparator(_natural_order_comparator)


    def then_comparing(
        self, 
        param: Union[Callable[[object], object], Comparator],
        key_comparator: Optional[Comparator]=None
    ) -> Comparator:
        if key_comparator is not None:
            return ThenComparingComparator(
                _base_comparator=self,
                _key_extractor=param,
                _key_comparator=key_comparator
            )

        if isinstance(param, Comparator):
            return ThenComparingComparator(
                _base_comparator=self,
                _key_extractor=_identity_func,
                _key_comparator=param
            )

        return ThenComparingComparator(
            _base_comparator=self,
            _key_extractor=param,
            _key_comparator=_natural_order_comparator
        )


    def reversed(self) -> Comparator:
        return ReversedOrderComparator(_base_comparator=self)


    def nullsFirst(self) -> Comparator:
        return NullComparator(_base_comparator=self, _null_first=True)


    def nullsLast(self) -> Comparator:
        return NullComparator(_base_comparator=self, _null_first=False)


    def compare(self, o1, o2) -> int:
        raise NotImplementedError("Do not raw-use `Compartor` class.")


    def equals(self, o1, o2) -> bool:
        return self.compare(o1, o2) == 0

    
class NaturalOrderComparator(Comparator):
    def compare(self, o1, o2) -> int:
        if o1 < o2: return -1
        if o1 > o2: return 1
        return 0


class ReversedOrderComparator(Comparator):
    def __init__(self, _base_comparator: Comparator) -> None:
        self._base_comparator = _base_comparator


    def compare(self, o1, o2) -> int:
        return -1 * self._base_comparator.compare(o1, o2)


class ThenComparingComparator(Comparator):
    def __init__(
        self,
        *,
        _base_comparator: Comparator,
        _key_extractor: Callable[[object], object],
        _key_comparator: Comparator,
    ) -> None:
        self._base_comparator = _base_comparator
        self._key_extractor = _key_extractor
        self._key_comparator = _key_comparator


    def compare(self, o1, o2) -> int:
        base_result = self._base_comparator.compare(o1, o2)
        if base_result != 0: return base_result

        return self._key_comparator.compare(self._key_extractor(o1), self._key_extractor(o2))
        
        
class KeyComparator(Comparator):
    def __init__(self, *, _key_extractor: Callable[[object], object], _key_comparator: Comparator) -> None:
        self._key_extractor = _key_extractor
        self._key_comparator = _key_comparator
    

    def compare(self, o1, o2) -> int:
        return self._key_comparator.compare(self._key_extractor(o1), self._key_extractor(o2))


class NullComparator(Comparator):
    def __init__(self, *, _base_comparator: Comparator, _null_first: bool) -> None:
        self._base_comparator = _base_comparator
        self._null_first = _null_first
    

    def compare(self, o1, o2) -> int:
        if o1 is None and o2 is None: return 0
        if o1 is None: return -1 if self._null_first else 1
        if o2 is None: return 1 if self._null_first else -1
        return self._base_comparator.compare(o1, o2)


_identity_func = lambda x: x
_natural_order_comparator = NaturalOrderComparator()

```