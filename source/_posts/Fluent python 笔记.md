---
title: Fluent python笔记
date: 2017-11-13 10:47:30
tags:
    - python
---

### An array of sequence


* 除了常见的mutable, immutable分类外，还有container, flat的分法, flat包括str, bytes, bytearray, memoryview, array.array, 这些都只能存原始数据类型, container包括list, tuple, collections.deque, 这些可以存其他类型

#### named tuple
``` python
from collections import namedtuple
City = namedtuple('City', 'name county population coordinates')
tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691677))
```

* tokyo
``` python
City(name='Tokyo', county='JP', population=36.933, coordinates=(35.689722, 139.1677))
```


#### deque
* 一般的序列中, pop和append都是操作最后进栈的数据(LIFO), deque可以从左或者从右操作, 类似于双向链表
``` python
from collections import deque
dq = deque(range(10), maxlen=10)
dq.append(10)
dq.extend([-1, -2])
dq.appendleft(-1)
dq.extendleft([-3, -4])
dq.roteate(-4)
```

可参考文档  https://docs.python.org/3.6/library/collections.html#collections.deque