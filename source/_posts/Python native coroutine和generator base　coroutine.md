---
title: Python native协程和generator base 协程
date: 2018-08-05 16:16:28
tags:
---

#### generator实现coroutine
``` python
def simple_coroutine():
    x = yield 10
    yield x

coro = simple_coroutine()
print(next(coro))
print(next.send(20))
```
> 可以通过yeild实现简单的协程，yeild可以pull generator的结果，send可以 push参数进generator,send的过程中，首先使coro恢复执行，然后传值给yeild，最终返回


#### asyncio.coroutine/yield from 和 async/await
> 这两种协程的方式没有功能上的区别，但是不能混用，只不过前者是给予generator的，后者是python3.5之后的原生协程。比如不能使用async/yield from。
> 但是这两种方式却可以同时使用, 即可以在asyncio.coroutine/yield中使用async/await，反之亦然。
``` python
import asyncio
import datetime
import random
import types


@types.coroutine
def my_sleep_func():
    yield from asyncio.sleep(random.randint(0, 5))


async def display_date(num, loop, ):
    end_time = loop.time() + 50.0
    while True:
        print("Loop: {} Time: {}".format(num, datetime.datetime.now()))
        if (loop.time() + 1.0) >= end_time:
            break
        await my_sleep_func()


loop = asyncio.get_event_loop()

asyncio.ensure_future(display_date(1, loop))
asyncio.ensure_future(display_date(2, loop))

loop.run_forever()
```
