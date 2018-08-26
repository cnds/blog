---
title: Python的传值方式
date: 2017-08-26 10:36:10
tags:
    - Python
---

#### Python的传值
> Python函数的传值分两种，对于不可变对象，是值传递(call by value)，对于可变对象，是引用传递(call by reference)
``` python
# 不可变对象
d = 1
def add(num):
    print id(num)
    num = num + 10
    print id(num)

add(d)
print d
# 输出:
# 140603518112776
# 140603518112776
# 140603518112536
# 1


# 可变对象
d = [1]
def change(num):
    print id(num)
    num.append(10)
    print id(num)

change(d)
print d
# 输出:
# 4431271840
# 4431271840
# 4431271840
# [1, 10]
```
> 上面的代码可以看到，对于不可变对象，在函数中改变了num的值并不会影响d, 同时num的id也发生了变化，这说明传到函数中的d的一个拷贝。


> 对于可变对象，在函数中改变num，外面的d也发生了改变，num的id并没有发生变化,这说明传到函数中的值是d的引用。