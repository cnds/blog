---
title: RESTful API 设计原则
date: 2018-07-29 22:39:00
tags:
    - tips
---

#### 错误类型
> 错误分为Errors和Faults两种
##### Errors
* 这种错误表示客户端传入了错误的数据，服务端正确的拒绝了客户端的数据，返4xx错误，Errors不会影响API的可用性
##### Faults
* 表示服务端的错误，客户端正确的传入了数据，但是服务端不能正确返回结果，返回5xx错误，Faults会影响API的可用性

#### 方法
> 除了常用的GET,PUT,POST,DELETE之外，还有一个PATCH方法（[RFC 5789](https://tools.ietf.org/html/rfc5789)），是比较容易被忽略的，很多情况下都被PUT取代了，但是在定义上，这两个方法还是不太一样的

|Method|Description|Is Idempotent|
|:----:|:---------:|:-----------:|
|GET|Return the current value of an object|True|
|PUT|Replace an object, or create a named object, when applcable|True|
|DELETE|Delete an object|True|
|POST|Create a new object based on the data provided, or submit a command|False|
|HEAD|Return metadata of an object of a GET response. Resources that support the GET method MAY support the HEAD method as well|True|
|PATCH|Apply a partial update to an object|False|
|OPTIONS|Get information about a request|True|

> 可以看到，PUT是修改一整个对象，PATCH只是修改对象的一部分，而且PUT是幂等的，PATCH不是。PATCH支持UPSERT语义，如果服务器支持PATCH的UPSERT语义，那么在接受到数据时，需要对资源进行判断，如果是一个不存在对资源，则此时相当于create，否则相当于update。
>
> 如果客户端想告诉服务端是否支持UPSERT，可以在HTTP headers中根据If-Match或If-None-Match来表示。如果有If-Match则不能支持create, 如果有If-None-Match: *，则不支持update。

#### 关键敏感信息
> 一般不建议将关键信息放在URL上，可以将这些信息通过header传递，但是并不是所有的客户端都支持这样做，所以需要根据实际情况考虑。
* 比如，设备的唯一识别号sn,如果放在URL上，就不是很安全，这时候可以考虑放在headers

#### 集合的处理
##### 路由的格式
> https://{serviceRoot}/{collection}/{id}

##### 大数据集合
> 一般大数据的集合，服务器都会采用分页来处理，在第一个集合中必须带有下一页都地址(nextLink)，这样既能保证每次单独使用API时数据量不大，又可以在处理需要取得所有数据的情况时，根据nextLink取得所有数据

#### 长时间运行的操作
> 有时候会碰到需要长时间计算才能有结果的请求，这时候就时长时间运行的操作，类似于异步
* 客户端发出请求后，服务端不能直接返回结果，需要返回HTTP/1.1 202 Accepted 作为结果，这个202的状态不带任何返回的body。headers中带Operation-Location表示实际正在操作的集合，这个location支持GET查询，可以查询到操作的状态(NotStarted|Running|Succeeded|Failed)，还可以提供操作开始结束时间，甚至完成的百分比，如果是完成状态，需要带上请求的resourceLocation。有需要的话，可以在headers带上Retry-After字段，提示客户端在一段时间后重新请求。
* 比如从 https://api.example.com/v1.0/datbases/db1 请求，得到202状态和Operation-Location(https://api.example.com/v1.0/operations/123)，客户端就可以查询合格Operation-Location得到操作的状态，如果是成功了，同时还会得到resourceLocation(https://api.example.com/v1.0/datbases/db1)，如果有Retry-After header，客户端在一段时间后再次请求Operation-Location
* 除了Retry-After方式之外，还有可以通过推送的方式通知客户端操作完成，这样需要客户端在请求操作的时候同时订阅消息。