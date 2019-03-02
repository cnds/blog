# Python Mock的细节

* Mock是Python unittest中的一个模块，用来模拟函数的返回值，但在mock某个函数时，需要注意，不同的函数有不同的表现

## 现象

#### 普通函数和静态函数

* 首先来看普通的函数

  ```python
  from unittest.mock import Mock
  
  
  def single_func(data):
      return 'single func: %s' % data
          
  if __name__ == '__main__':
  
      print('<<<==========================FUNCTION======================>>>')
      rs1 = single_func('no mock 1')
      rs2 = single_func('no mock 2')
      print('origin func 1: %s, id: %s' % (rs1, id(rs1)))
      print('origin func 2: %s, id: %s' % (rs2, id(rs2)))
      print('origin func property: %s' % single_func.__dict__)
      single_func = Mock(return_value='mocked')
      print('mocked func property: %s' % single_func.__dict__)
      print('mocked func1: %s, id: %s' % (single_func('no mock 1'), id(single_func('no mock 1'))))
      print('mocked func2: %s, id: %s' % (single_func('no mock 2'), id(single_func('no mock 2'))))
  ```

  

* 以上代码mock了`single_func`这个函数，运行结果如下

  ```shell
  <<<==========================FUNCTION======================>>>
  origin func 1: single func: no mock 1, id: 2590139904048
  origin func 2: single func: no mock 2, id: 2590139904120
  origin func property: {}
  mocked func property: {'_mock_return_value': 'mocked', '_mock_parent': None, '_mock_name': None, '_mock_new_name': '', '_mock_new_parent': None, '_mock_sealed': False, '_spec_class': None, '_spec_set': None, '_spec_signature': None, '_mock_methods': None, '_mock_children': {}, '_mock_wraps': None, '_mock_delegate': None, '_mock_called': False, '_mock_call_args': None, '_mock_call_count': 0, '_mock_call_args_list': [], '_mock_mock_calls': [], 'method_calls': [], '_mock_unsafe': False, '_mock_side_effect': None}
  mocked func1: mocked, id: 2590132817456
  mocked func2: mocked, id: 2590132817456
  ```

  

* 这里面有几个细节：

  *  首先在没有mock之前，函数的两次调用结果和内存地址是不同的
  * mock之后，两次调用的结果就相同了，而且函数的属性也发生了变化```(single_func.__dict__)```
  * 两次运行不仅结果相同，而且内存地址也相同

* 同样地，静态方法的运行结果如下，和普通函数没有什么区别：

  ```python
  from unittest.mock import Mock
  
  
  class StaticClass(object):
  
      @staticmethod
      def static_func(data):
          return 'static func: %s' % data
  
          
  if __name__ == '__main__':
  
      print('<<<==========================STATICMETHOD======================>>>')
      rs1 = StaticClass.static_func('no mock 1')
      rs2 = StaticClass.static_func('no mock 2')
      print('origin func 1: %s, id: %s' % (rs1, id(rs1)))
      print('origin func 2: %s, id: %s' % (rs2, id(rs2)))
      print('origin func property: %s' % StaticClass.static_func.__dict__)
      StaticClass.static_func = Mock(return_value='mocked')
      print('mocked func property: %s' % StaticClass.static_func.__dict__)
      rs3 = StaticClass.static_func('no mock 3')
      rs4 = StaticClass.static_func('no mock 4')
      print('mocked func1: %s, id: %s' % (rs3, id(rs4)))
      print('mocked func2: %s, id: %s' % (rs4, id(rs4)))
  
  ```

  ```shell
  origin func 1: static func: no mock 1, id: 2472839443488
  origin func 2: static func: no mock 2, id: 2472839430056
  origin func property: {}
  mocked func property: {'_mock_return_value': 'mocked', '_mock_parent': None, '_mock_name': None, '_mock_new_name': '', '_mock_new_parent': None, '_mock_sealed': False, '_spec_class': None, '_spec_set': None, '_spec_signature': None, '_mock_methods': None, '_mock_children': {}, '_mock_wraps': None, '_mock_delegate': None, '_mock_called': False, '_mock_call_args': None, '_mock_call_count': 0, '_mock_call_args_list': [], '_mock_mock_calls': [], 'method_calls': [], '_mock_unsafe': False, '_mock_side_effect': None}
  mocked func1: mocked, id: 2472832355888
  mocked func2: mocked, id: 2472832355888
  ```

  

#### 单例模式下的函数

* 先看代码

  ```python
  from unittest.mock import Mock
  
  
  class SingletonClass(object):
  
      def __new__(cls, *args, **kwargs):
          if not hasattr(cls, '_instance'):
              origin = super(SingletonClass, cls)
              cls._instance = origin.__new__(cls, *args, **kwargs)
  
          return cls._instance
  
      def singleton_func(self, data):
          return 'singleton func: %s' % data
  
          
  if __name__ == '__main__':
  
      print('<<<==========================SINGLETON FUNCTION======================>>>')
      singleton_cls = SingletonClass()
      rs1 = singleton_cls.singleton_func('no mock 1')
      rs2 = singleton_cls.singleton_func('no mock 2')
      print('origin func 1: %s, id: %s' % (rs1, id(rs1)))
      print('origin func 2: %s, id: %s' % (rs2, id(rs2)))
      print('origin func property: %s' % StaticClass.static_func.__dict__)
      singleton_cls.singleton_func = Mock(return_value='mocked')
      print('mocked func property: %s' % singleton_cls.singleton_func.__dict__)
      rs3 = SingletonClass().singleton_func('no mock 3')
      rs4 = SingletonClass().singleton_func('no mock 4')
      print('mocked func1: %s, id: %s' % (rs3, id(rs3)))
      print('mocked func2: %s, id: %s' % (rs4, id(rs4)))
  
  ```

* 结果

  ``` shell
  origin func 1: singleton func: no mock 1, id: 2472839479744
  origin func 2: singleton func: no mock 2, id: 2472839479824
  origin func property: {'_mock_return_value': 'mocked', '_mock_parent': None, '_mock_name': None, '_mock_new_name': '', '_mock_new_parent': None, '_mock_sealed': False, '_spec_class': None, '_spec_set': None, '_spec_signature': None, '_mock_methods': None, '_mock_children': {}, '_mock_wraps': None, '_mock_delegate': None, '_mock_called': True, '_mock_call_args': call('no mock 4'), '_mock_call_count': 2, '_mock_call_args_list': [call('no mock 3'), call('no mock 4')], '_mock_mock_calls': [call('no mock 3'), call('no mock 4')], 'method_calls': [], '_mock_unsafe': False, '_mock_side_effect':
  None}
  mocked func property: {'_mock_return_value': 'mocked', '_mock_parent': None, '_mock_name': None, '_mock_new_name': '', '_mock_new_parent': None, '_mock_sealed': False, '_spec_class': None, '_spec_set': None, '_spec_signature': None, '_mock_methods': None, '_mock_children': {}, '_mock_wraps': None, '_mock_delegate': None, '_mock_called': False, '_mock_call_args': None, '_mock_call_count': 0, '_mock_call_args_list': [], '_mock_mock_calls': [], 'method_calls': [], '_mock_unsafe': False, '_mock_side_effect': None}
  mocked func1: mocked, id: 2472832355888
  mocked func2: mocked, id: 2472832355888
  ```

* 与普通函数和静态函数相比，尽管mock之后的函数调用采用了重新创建实例的方式，由于单例模式的特点，结果并没有区别，还是被成功的mock



#### 普通类的函数

* 代码

  ```shell
  from unittest.mock import Mock
  
  
  class NormalClass(object):
      
      def normal_func(self, data):
          return 'normal func %s' % data
  
          
  if __name__ == '__main__':
  
      print('<<<==========================NORMAL FUNCTION======================>>>')
      normal_cls = NormalClass()
      rs1 = normal_cls.normal_func('no mock 1')
      rs2 = normal_cls.normal_func('no mock 2')
      print('origin func 1: %s, id: %s' % (rs1, id(rs1)))
      print('origin func 2: %s, id: %s' % (rs2, id(rs2)))
      print('origin func property: %s' % normal_cls.normal_func.__dict__)
      normal_cls.normal_func = Mock(return_value='mocked')
      print('mocked func property: %s' % normal_cls.normal_func.__dict__)
      rs3 = NormalClass().normal_func('no mock 3')
      rs4 = NormalClass().normal_func('no mock 4')
      print('mocked func1: %s, id: %s' % (rs3, id(rs3)))
      print('mocked func2: %s, id: %s' % (rs4, id(rs4)))
  ```

* 结果

  ```shell
  origin func 1: normal func no mock 1, id: 2308646795496
  origin func 2: normal func no mock 2, id: 2308646795568
  origin func property: {}
  mocked func property: {'_mock_return_value': 'mocked', '_mock_parent': None, '_mock_name': None, '_mock_new_name': '', '_mock_new_parent': None, '_mock_sealed': False, '_spec_class': None, '_spec_set': None, '_spec_signature': None, '_mock_methods': None, '_mock_children': {}, '_mock_wraps': None, '_mock_delegate': None, '_mock_called': False, '_mock_call_args': None, '_mock_call_count': 0, '_mock_call_args_list': [], '_mock_mock_calls': [], 'method_calls': [], '_mock_unsafe': False, '_mock_side_effect': None}
  mocked func1: normal func no mock 3, id: 2308646796000
  mocked func2: normal func no mock 4, id: 2308646796216
  mocked func3: mocked, id: 2308639706672
  ```

* 与单例模式的方式一样，采用了重新创建实例的方式，这次的结果并没有被mock



## 原理

* Mock实际上是利用了[Monkey Patch](https://en.wikipedia.org/wiki/Monkey_patch)的特性，在运行时替换了函数

* 自己的简单Mock，仅实现了return_value

  ```python
  class MySimpleMock(object):
  
      def __init__(self, return_value):
          self.return_value = return_value
  
      def __call__(self, return_value):
          return self.return_value
     
  class SingletonClass(object):
  
      def __new__(cls, *args, **kwargs):
          if not hasattr(cls, '_instance'):
              origin = super(SingletonClass, cls)
              cls._instance = origin.__new__(cls, *args, **kwargs)
  
          return cls._instance
  
      def singleton_func(self, data):
          return 'singleton func: %s' % data
      
      
  if __name__ == '__main__':
      singleton_cls = SingletonClass()
      singleton_cls.singleton_func = MySimpleMock(return_value='my simple mock')
      rs7 = singleton_cls.singleton_func('test simple mock')
      rs8 = singleton_cls.singleton_func('test simple mock')
      print('mocked func 4: %s, id: %s' % (rs7, id(rs7)))
      print('mocked func 5: %s, id: %s' % (rs8, id(rs8)))
  ```

* 运行结果

  ```shell
  mocked func 4: my simple mock, id: 1571113957680
  mocked func 5: my simple mock, id: 1571113957680
  ```

* 陷阱

  * 在mock函数、静态函数、单例类函数时，由于全局只有一个实例，会导致一旦mock之后，代码中所有调用该函数的值都变成mock后的值

  

* 解决方案：使用patch

  ```python
  from unittest.mock import Mock, patch
      
      
  class SingletonClass(object):
  
      def __new__(cls, *args, **kwargs):
          if not hasattr(cls, '_instance'):
              origin = super(SingletonClass, cls)
              cls._instance = origin.__new__(cls, *args, **kwargs)
  
          return cls._instance
  
      def singleton_func(self, data):
          return 'singleton func: %s' % data
      
  if __name__ == '__main__':
      singleton_cls = SingletonClass()
  
      print('===============CONTEXT============')
      print(singleton_cls.singleton_func('no patch'))
      with patch('__main__.SingletonClass.singleton_func') as patched_func:
          patched_func.return_value = 'patched'
          print(singleton_cls.singleton_func('no patch'))
      print(singleton_cls.singleton_func('no patch'))
  
      print('===============START&STOP============')
      p = patch('__main__.SingletonClass.singleton_func', return_value='patched')
      p.start()
      print(singleton_cls.singleton_func('no patch'))
      p.stop()
      print(singleton_cls.singleton_func('no patch'))
  ```

  ```shell
  ===============CONTEXT============
  singleton func: no patch
  patched
  singleton func: no patch
  ===============START&STOP============
  patched
  singleton func: no patch
  ```

* 可以看到，不论使用哪种方法，之前被patch掉的函数还会恢复到正常的值



##  总结

* Mock本质上是monkey patch
* 全局唯一的对象，被mock后会影响到全局调用
* 使用patch 可以避免全局调用，可以使用上下文或者 start&stop