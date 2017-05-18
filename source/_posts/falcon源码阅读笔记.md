---
title: falcon-1.1.0源码阅读笔记
date: 2017-05-10 11:32:19
tags: 
    - 源码笔记
    - Python
---

##  Falcon API class

### \__init__
#### 构造类的时候，可传参数有meida_type, request_type, response_type, middleware, router,
* media_type的默认值为aapplication/json; charset=UTF-8
* request_type默认为Request类
* response_type默认为Response类
* middleware和router默认为None

#### middleware传入后,会通过prepare_middleware做处理, prepare_middleware是从api_helpers中导入的方法,返回的值为list。

#### prepare_middleware:
* 如果middleware为None，则返回空list
* 如果middleware不是list, 则将其转换为list([middleware])
* 检查middleware中的元素,是否有process_request, process_resource, process_response方法，如果没有其中任何一个,则返回错误, 也就是说middleware类中必须至少有这三种方法之一
* 如果有process_response方法,还要检查参数是否有req_succeeded
* 最后将三个方法组合成tuple添加到prepared_middleware中，返回。

#### prepare_middleware方法返回的值格式为[(process_request, process_resource, process_response), (...)]

###  \__call__

####  根据PEP3333， 构造一个callable的API实例，参数env是WSGI环境，是dict类型， start_response是一个callable function， req为Request的实例， resp为Response的实例

#### 遍历self._middleware， 首先将process_request和process_response拿出来(line 195)
* 如果process_request非空，则先执行该方法
* 如果process_response非空，则将其放入mw_pr_stack这个list中

#### 通过_get_responder方法拿到responder， 即该实例中的on_get, on_post之类的方法

#### 然后继续遍历self._middleware，如果有process_resource方法，先执行该方法，然后执行responder(req, resp, **params)，即执行了on_get,on_post,on_put这类HTTP请求方法

#### 执行完后，将req_succeeded更改为True（默认值为False)

#### 最后执行已经放入mw_pr_stack中的process_response方法
* 通过while方法遍历，用pop方法依次取出方法后执行
* 如果执行失败，则将req_succeeded修改为False

#### 整个API实例的执行过程，通过try, except, else, finally来完成
````python
try:
	process_request
else:
	process_resource
	responder
finally:
	process_response
````

* 可以看到整个执行顺序如上

#### 通过_get_body方法获取resp中的运行结果body，设置好status和headers，整个请求执行完成
