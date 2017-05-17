---
title: falcon-1.1.0源码笔记II
date: 2017-05-16 10:39:37
tags:
    - 源码笔记
    - python
---

### add_route(self, uri_template, resource, \*args, \**kwargs)

添加路由，这个方法的参数，uri_template就是路由的路径，resource是处理请求的responder的类的实例，一般responder是on_get,on_post等方法

#### 具体的实现方法：
先判断uri_template是否格式是否规范（必须是string格式，以‘/’开头，且不能包含'//'）
通过routing中的create_http_method_map方法，获得method_map(dict类型）
* 如果http_method在resource中有定义，就在method_map中添加相应的键值对，格式{'GET': 'on_get()'}
* 对于OPTIONS方法，如果resource中没有定义，则生成一个默认的格式
* 其余的未定义方法，统一生成method_not_allowed格式
* 生成方法都在responder.py中有定义
通过_router.add_route将生成的method_map与uri_template,resource一起添加到new_node 
* 具体方法在routing　CompileRouter中(待补充）

### add_sink(self, sink, prefix=r'/')
### add_error_handler(self, exception, handler=None)
### set_error_serializer(self, serializer)