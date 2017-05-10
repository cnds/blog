---
title: falcon源码阅读笔记
date: 2017-05-10 11:32:19
tags:
---

###  Falcon API class

构造类的时候，可传参数有meida_type, request_type, response_type, middleware, router,
* media_type的默认值为aapplication/json; charset=UTF-8
* request_type默认为Request类
* response_type默认为Response类
* middleware和router默认为None

middleware传入后,会通过prepare_middleware做处理, prepare_middleware是从api_helpers中导入的方法,返回的值为list。

prepare_middleware:
* 如果middleware为None，则返回空list
* 如果middleware不是list, 则将其转换为list([middleware])
* 检查middleware中的元素,是否有process_request, process_resource, process_response方法，如果没有其中任何一个,则返回错误, 也就是说middleware类中必须至少有这三种方法之一
* 如果有process_response方法,还要检查参数是否有req_succeeded
* 最后将三个方法组合成tuple添加到prepared_middleware中，返回。

prepare_middleware方法返回的值格式为[(process_request, process_resource, process_response), (...)]

