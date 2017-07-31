---
title: falcon-1.1.0源码阅读笔记V
date: 2017-05-23 10:36:10
tags:
	- Python
	- 源码笔记
---

## Hook decorators

### falcon有两个修饰器，before和after，顾名思义，就是分别在responder运行之前和之后运行的

### before
#### 这个修饰器实际是一个闭包，将action通过闭包传到内部函数_before
* action(req, resp, resource, params)
* _before首先或者responder
* 然后通过_wrap_with_before函数处理action和responder
* _wrap_with_before函数也是一个闭包，按将action函数传给shim，然后按顺序运行shim, responder
* 然后将resource里的responder设置成_wrap_with_before的返回值
````python
setattr(resource, responder_name, do_before_all)
````

### after
#### 和before类似，只是处理顺序变成了先responder后shim
* action(req, response, resource)