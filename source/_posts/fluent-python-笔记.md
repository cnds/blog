---
title: fluent python 笔记
date: 2017-11-13 10:47:30
tags:
---

## 第二章

### 除了常见的mutable, immutable分类外，还有container, flat的分法, flat包括str, bytes, bytearray, memoryview, array.array, 这些都只能存原始数据类型, container包括list, tuple, collections.deque, 这些可以存其他类型

### named tuple
``` python
from collections import namedtuple
City = namedtuple('City', 'name county population coordinates')
tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691677))
```

#### tokyo
``` python
City(name='Tokyo', county='JP', population=36.933, coordinates=(35.689722, 139.1677))
```