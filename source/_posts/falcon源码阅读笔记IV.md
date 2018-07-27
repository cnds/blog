---
title: falcon-1.1.0源码阅读笔记IV
date: 2017-05-22 10:41:02
tags:
    - Python
    - 源码笔记
---

### Response class

#### \__init__
```` python
    def __init__(self):
        self.status = '200 OK'
        self._headers = {}

        # NOTE(tbug): will be set to a SimpleCookie object
        # when cookie is set via set_cookie
        self._cookies = None

        self.body = None
        self.data = None
        self.stream = None
        self.stream_len = None

        if self.context_type is None:
            # PERF(kgriffs): The literal syntax is more efficient than dict().
            self.context = {}
        else:
            self.context = self.context_type()
````

可以看到，在默认的状态码为200 OK，默认的_headers为{}

##### 此外，还有一些方法也被放在了这个类中，比如set_headers，get_header， set_cookie等

- 注意：跟headers有关的操作，比如set_header等，headers全部都是小写
- 如果预设的headers里有大写，比如Content-Type:application/json，如果想在返回中改为别的格式，由于set_header会将字母全部转化为小写，覆盖会失败，也就是修改不会成功


#### 再回头看API class 中的\__call__函数
````python
if req.method == 'HEAD' or resp.status in self._BODILESS_STATUS_CODES:
            body = []
        else:
            body, length = self._get_body(resp, env.get('wsgi.file_wrapper'))
            if length is not None:
                resp._headers['content-length'] = str(length)
````

里面_get_body的实现


````python
 def _get_body(self, resp, wsgi_file_wrapper=None):
        body = resp.body
        if body is not None:
            if not isinstance(body, bytes):
                body = body.encode('utf-8')

            return [body], len(body)

        data = resp.data
        if data is not None:
            return [data], len(data)

        stream = resp.stream
        if stream is not None:
            if hasattr(stream, 'read'):
                if wsgi_file_wrapper is not None:
                    iterable = wsgi_file_wrapper(stream,
                                                 self._STREAM_BLOCK_SIZE)
                else:
                    iterable = iter(
                        lambda: stream.read(self._STREAM_BLOCK_SIZE),
                        b''
                    )
            else:
                iterable = stream
            return iterable, resp.stream_len

        return [], 0
````
可以看到，这个方法就是把resp中的值放到body中
在responder执行的时候，只需要在最后将执行的结果写入resp.body或者resp.data中，就可以达到返回结果的作用