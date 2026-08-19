title: Python奇技淫巧
date: 2015-11-14 00:42:10
tags: Python
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


## 显示有限的接口到外部

当发布python第三方package时, 并不希望代码中所有的函数或者class可以被外部import, 在`__init__.py`中添加`__all__`属性, 
该list中填写可以import的类或者函数名, 可以起到限制的import的作用, 防止外部import其他函数或者类

<!--more-->

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from base import APIBase
from client import Client
from decorator import interface, export, stream
from server import Server
from storage import Storage
from util import (LogFormatter, disable_logging_to_stderr,
                       enable_logging_to_kids, info)

__all__ = ['APIBase', 'Client', 'LogFormatter', 'Server',
           'Storage', 'disable_logging_to_stderr', 'enable_logging_to_kids',
           'export', 'info', 'interface', 'stream']
```

## with的魔力

with语句需要支持`上下文管理协议的对象`, 上下文管理协议包含`__enter__`和`__exit__`两个方法. with语句建立运行时上下文需要通过这两个方法执行`进入和退出`操作.

其中`上下文表达式`是跟在with之后的表达式, 该表示大返回一个上下文管理对象

```
# 常见with使用场景
with open("test.txt", "r") as my_file:  # 注意, 是__enter__()方法的返回值赋值给了my_file,
    for line in my_file:
        print line
```

详细原理可以查看这篇文章, [浅谈 Python 的 with 语句](https://www.ibm.com/developerworks/cn/opensource/os-cn-pythonwith/)

知道具体原理, 我们可以自定义支持上下文管理协议的类, 类中实现`__enter__`和`__exit__`方法

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

class MyWith(object):

    def __init__(self):
        print "__init__ method"

    def __enter__(self):
        print "__enter__ method"
        return self  # 返回对象给as后的变量

    def __exit__(self, exc_type, exc_value, exc_traceback):
        print "__exit__ method"
        if exc_traceback is None:
            print "Exited without Exception"
            return True
        else:
            print "Exited with Exception"
            return False

def test_with():
    with MyWith() as my_with:
        print "running my_with"
    print "------分割线-----"
    with MyWith() as my_with:
        print "running before Exception"
        raise Exception
        print "running after Exception"


if __name__ == '__main__':
    test_with()
```

执行结果如下:

```
__init__ method
__enter__ method
running my_with
__exit__ method
Exited without Exception
------分割线-----
__init__ method
__enter__ method
running before Exception
__exit__ method
Exited with Exception
Traceback (most recent call last):
  File "bin/python", line 34, in <module>
    exec(compile(__file__f.read(), __file__, "exec"))
  File "test_with.py", line 33, in <module>
    test_with()
  File "test_with.py", line 28, in test_with
    raise Exception
Exception
```

> 证明了会先执行`__enter__`方法, 然后调用with内的逻辑, 最后执行`__exit__`做退出处理, 并且, 即使出现异常也能正常退出


## filter的用法

相对`filter`而言, map和reduce使用的会更频繁一些, `filter`正如其名字, 按照某种规则`过滤`掉一些元素

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

lst = [1, 2, 3, 4, 5, 6]
# 所有奇数都会返回True, 偶数会返回False被过滤掉
print filter(lambda x: x % 2 != 0, lst)

#输出结果
[1, 3, 5]
```


## 一行作判断

当条件满足时, 返回的为等号后面的变量, 否则返回else后语句

```
lst = [1, 2, 3]
new_lst = lst[0] if lst is not None else None
print new_lst

# 打印结果
1
```

## 装饰器之单例

> 使用装饰器实现简单的单例模式

```
# 单例装饰器
def singleton(cls):
    instances = dict()  # 初始为空
    def _singleton(*args, **kwargs):
        if cls not in instances:  #如果不存在, 则创建并放入字典
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return _singleton

@singleton
class Test(object):
    pass

if __name__ == '__main__':
    t1 = Test()
    t2 = Test()
    # 两者具有相同的地址
    print t1, t2
```



## staticmethod装饰器

类中两种常用的装饰, 首先区分一下他们

- 普通成员函数, 其中第一个隐式参数为`对象`
- `classmethod装饰器`, 类方法(给人感觉非常类似于OC中的类方法), 其中第一个隐式参数为`类`
- `staticmethod装饰器`, 没有任何隐式参数. `python中的静态方法类似与C++中的静态方法`


```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

class A(object):

    # 普通成员函数
    def foo(self, x):
        print "executing foo(%s, %s)" % (self, x)

    @classmethod   # 使用classmethod进行装饰
    def class_foo(cls, x):
        print "executing class_foo(%s, %s)" % (cls, x)

    @staticmethod  # 使用staticmethod进行装饰
    def static_foo(x):
        print "executing static_foo(%s)" % x

def test_three_method():
    obj = A()
    # 直接调用噗通的成员方法
    obj.foo("para")  # 此处obj对象作为成员函数的隐式参数, 就是self
    obj.class_foo("para")  # 此处类作为隐式参数被传入, 就是cls
    A.class_foo("para")  #更直接的类方法调用
    obj.static_foo("para")  # 静态方法并没有任何隐式参数, 但是要通过对象或者类进行调用
    A.static_foo("para")

if __name__ == '__main__':
    test_three_method()
    
# 函数输出
executing foo(<__main__.A object at 0x100ba4e10>, para)
executing class_foo(<class '__main__.A'>, para)
executing class_foo(<class '__main__.A'>, para)
executing static_foo(para)
executing static_foo(para)
```

## property装饰器


- 定义私有类属性

将`property`与装饰器结合实现属性私有化(`更简单安全的实现get和set方法`)

```python
#python内建函数
property(fget=None, fset=None, fdel=None, doc=None)
```

`fget`是获取属性的值的函数,`fset`是设置属性值的函数,`fdel`是删除属性的函数,`doc`是一个字符串(like a comment).从实现来看，这些参数都是可选的

property有三个方法`getter()`, `setter()`和`delete()` 来指定fget, fset和fdel。 这表示以下这行

```
class Student(object):

    @property  #相当于property.getter(score) 或者property(score)
    def score(self):
        return self._score

    @score.setter #相当于score = property.setter(score)
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
```



## __iter__魔法

- 通过yield和`__iter__`的结合, 我们可以把一个对象变成可迭代的
- 通过`__str__`的重写, 可以直接通过想要的形式打印对象


```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

class TestIter(object):

    def __init__(self):
        self.lst = [1, 2, 3, 4, 5]

    def read(self):
        for ele in xrange(len(self.lst)):
            yield ele

    def __iter__(self):
        return self.read()

    def __str__(self):
        return ','.join(map(str, self.lst))
    
    __repr__ = __str__

def test_iter():
    obj = TestIter()
    for num in obj:
        print num
    print obj

if __name__ == '__main__':
    test_iter()
```


## 神奇partial

partial使用上很像C++中仿函数(函数对象).

**在stackoverflow给出了类似与partial的运行方式**

```
def partial(func, *part_args):
    def wrapper(*extra_args):
        args = list(part_args)
        args.extend(extra_args)
        return func(*args)

    return wrapper
```

利用用闭包的特性绑定预先绑定一些函数参数, 返回一个可调用的变量, 直到真正的调用执行

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from functools import partial

def sum(a, b):
    return a + b

def test_partial():
    fun = partial(sum, 2)   # 事先绑定一个参数, fun成为一个只需要一个参数的可调用变量
    print fun(3)  # 实现执行的即是sum(2, 3)

if __name__ == '__main__':
    test_partial()
    
# 执行结果
5
```





## 神秘eval

eval我理解为一种内嵌的python解释器(这种解释可能会有偏差), 会解释字符串为对应的代码并执行, 并且将执行结果返回

看一下下面这个例子

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

def test_first():
    return 3

def test_second(num):
    return num

action = {  # 可以看做是一个sandbox
        "para": 5,
        "test_first" : test_first,
        "test_second": test_second
        }

def test_eavl():  
    condition = "para == 5 and test_second(test_first) > 5"
    res = eval(condition, action)  # 解释condition并根据action对应的动作执行
    print res

if __name__ == '_
```

## exec

- exec在Python中会忽略返回值, 总是返回None, eval会返回执行代码或语句的返回值
- `exec`和`eval`在执行代码时, 除了返回值其他行为都相同
- 在传入字符串时, 会使用`compile(source, '<string>', mode)`编译字节码. mode的取值为`exec`和`eval`


```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

def test_first():
    print "hello"

def test_second():
    test_first()
    print "second"

def test_third():
    print "third"

action = {
        "test_second": test_second,
        "test_third": test_third
        }

def test_exec():
    exec "test_second" in action

if __name__ == '__main__':
    test_exec()  # 无法看到执行结果
```


## getattr

`getattr(object, name[, default])`Return the value of the named attribute of object. name must be a string. If the string is the name of one of the object’s attributes, the result is the value of that attribute. For example, getattr(x, 'foobar') is equivalent to x.foobar. If the named attribute does not exist, default is returned if provided, otherwise AttributeError is raised.

通过string类型的name, 返回对象的name属性(方法)对应的值, 如果属性不存在, 则返回默认值, 相当于object.name



```
# 使用范例
class TestGetAttr(object):

    test = "test attribute"

    def say(self):
        print "test method"

def test_getattr():
    my_test = TestGetAttr()
    try:
        print getattr(my_test, "test")
    except AttributeError:
        print "Attribute Error!"
    try:
        getattr(my_test, "say")()
    except AttributeError: # 没有该属性, 且没有指定返回值的情况下
        print "Method Error!"

if __name__ == '__main__':
    test_getattr()
    
# 输出结果
test attribute
test method
```


## 命令行处理

```py
def process_command_line(argv):
    """
    Return a 2-tuple: (settings object, args list).
    `argv` is a list of arguments, or `None` for ``sys.argv[1:]``.
    """
    if argv is None:
        argv = sys.argv[1:]

    # initialize the parser object:
    parser = optparse.OptionParser(
        formatter=optparse.TitledHelpFormatter(width=78),
        add_help_option=None)

    # define options here:
    parser.add_option(      # customized description; put --help last
        '-h', '--help', action='help',
        help='Show this help message and exit.')

    settings, args = parser.parse_args(argv)

    # check number of arguments, verify values, etc.:
    if args:
        parser.error('program takes no command-line arguments; '
                     '"%s" ignored.' % (args,))

    # further process settings & args if necessary

    return settings, args

def main(argv=None):
    settings, args = process_command_line(argv)
    # application code here, like:
    # run(settings, args)
    return 0        # success

if __name__ == '__main__':
    status = main()
    sys.exit(status)
```

## 读写csv文件

```
# 从csv中读取文件, 基本和传统文件读取类似
import csv
with open('data.csv', 'rb') as f:
    reader = csv.reader(f)
    for row in reader:
        print row
# 向csv文件写入
import csv
with open( 'data.csv', 'wb') as f:
    writer = csv.writer(f)
    writer.writerow(['name', 'address', 'age'])  # 单行写入
    data = [
            ( 'xiaoming ','china','10'),
            ( 'Lily', 'USA', '12')]
    writer.writerows(data)  # 多行写入
```

## 各种时间形式转换

**只发一张网上的图, 然后差文档就好了, 这个是记不住的**

![记不住](http://ww1.sinaimg.cn/large/ab508d3djw1exykn9rbe1j20kj0f977t.jpg)



## 字符串格式化

一个非常好用, 很多人又不知道的功能

```
>>> name = "andrew"
>>> "my name is {name}".format(name=name)
'my name is andrew'
```

## 参考链接

- [What is the difference between @staticmethod and @classmethod in Python?](http://stackoverflow.com/questions/136097/what-is-the-difference-between-staticmethod-and-classmethod-in-python)
- [Python @property versus getters and setters](http://stackoverflow.com/questions/6618002/python-property-versus-getters-and-setters)
- [How does the @property decorator work?](http://stackoverflow.com/questions/17330160/how-does-the-property-decorator-work)
- [How does the functools partial work in Python?](http://stackoverflow.com/questions/15331726/how-does-the-functools-partial-work-in-python)
- [What's the difference between eval, exec, and compile in Python?](http://stackoverflow.com/questions/2220699/whats-the-difference-between-eval-exec-and-compile-in-python)
- [Be careful with exec and eval in Python](http://lucumr.pocoo.org/2011/2/1/exec-in-python/)
- [Python (and Python C API): __new__ versus __init__](http://stackoverflow.com/questions/4859129/python-and-python-c-api-new-versus-init/)
- [Python 'self' keyword](http://stackoverflow.com/questions/6019627/python-self-keyword) `self`不是关键字, 是一个约定的变量名
- [Python进阶必读汇总](http://www.dongwm.com/)
- [使python类可以判断真值](http://python.net/~goodger/projects/pycon/2007/idiomatic/handout.html#truth-values)
- [Best Python Resources](http://www.fullstackpython.com/best-python-resources.html)
- [Python安全编码指南](http://drops.wooyun.org/tips/10383)
