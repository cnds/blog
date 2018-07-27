---
title: falcon-1.1.0源码阅读笔记III
date: 2017-05-18 10:39:15
tags:
    - Python
    - 源码笔记
---

### Request class

#### \__init__(self, env, options=None)

实例化类的时候，需要传入env参数(dict类型)，具体可参考PEP3333
不传入options参数的时候，options为RequestOptions类，这个类里实际就是几个默认设置
```` python
    __slots__ = (
        'keep_blank_qs_values',
        'auto_parse_form_urlencoded',
        'auto_parse_qs_csv',
        'strip_url_path_trailing_slash',
    )

    def __init__(self):
        self.keep_blank_qs_values = False
        self.auto_parse_form_urlencoded = False
        self.auto_parse_qs_csv = True
        self.strip_url_path_trailing_slash = True
````
##### 这些设置在后续处理env中的参数时有用到，比如self.query_string = env['QUERY_STRING']，如果有query_string，就需要通过parse_query_string方法解析
* parse_query_string(query_string, keep_blank_qs_values=False, parse_qs_csv)
* 可以看到这里的设置代码
````python
        try:
            self.query_string = env['QUERY_STRING']
        except KeyError:
            self.query_string = ''
            self._params = {}
        else:
            if self.query_string:
                self._params = parse_query_string(
                    self.query_string,
                    keep_blank_qs_values=self.options.keep_blank_qs_values,
                    parse_qs_csv=self.options.auto_parse_qs_csv,
                )

            else:
                self._params = {}
````
这里将options中的设置传入parse_query_string方法，得到一个字典

##### 通过header_property将HTTP header设置成属性，另外也添加了一些属性，比如client_accept_json(通过方法client_accepts), params(self._params)等

##### 最后还有一些自定义的方法，用来获取query_string里的参数，比如get_param,get_param_as_int等

##### 可以看到，整个Request class主要是存储了http请求的信息，并且定义成为自身属性(property)，在API class调用的时候，这些参数可以保证每次请求的独立性，即每次请求都有一个单独的req

##### 每个请求的req各不相同，在API class中 \__call__运行的时候，通过_get_responder获得的responder， params, resource, uri_template也不相同