title: Python闭包和装饰器
date: 2014-12-31 13:24:02
tags: Python
---

> 测试版本python2.7.8

#1. 闭包

> 闭包是词法闭包的简称, 是引用了自由变量的函数. 这个被引用的自由变量和这个函数一同存在, 即使已经离开了创造它的环境也不例外. 所以有另一种说法认为闭包是由函数和其他相关的引用环境组合而成的实体. 闭包在运行时可以有多个实例, 不同的引用环境和函数组合可以产生不同的实例(维基百科)

由上面这句话可以得到很多信息, 我们来解析一下:

- 闭包是一个函数
- 这个函数引用了自由变量
- 闭包有一个创建它的环境, 并能够脱离创建环境存在

<!--more-->

> 在python中, 当你调用一个函数A, 函数A传入了一个`自由变量`给函数A内的函数B, 函数B被称为闭包, 这个被传入的自由变量可以是变量或者函数等.


看一下例子

```python
# -*- coding: utf-8 -*-
#!/usr/bin/env python

from datetime import datetime

def print_now(fn) :
    def fn_wrap(*args, **kwargs) :
        print "now time: %s, the fn name: %s" % (datetime.now(), fn.__name__)
        return fn(*args, **kwargs) #执行fn函数, 并返回函数结果
    return fn_wrap


def hello(name = "andrew") :
    print "Hello World! %s" % name

if __name__ == '__main__':
    temp = print_now(hello) 
    temp() #fn_wrap在生命周期结束依然可以被调用
    temp("liu")
```

打印结果

```python
now time: 2014-12-30 21:53:39.518521, the fn name: hello
Hello World! andrew
now time: 2014-12-30 21:53:39.518723, the fn name: hello
Hello World! liu
```

这里`print_now`就相当于函数A, 函数A常被称为`工厂函数`, fn_wrap被称为`闭包`(函数B), fn被称为自由变量, `temp =print_now(hello)`这一句调用结束后,函数A返回一个闭包 `print_now`生命周期结束(`闭包(函数B)脱离创建环境继续存在`), 我们可以继续调用`temp()`, 来执行其中而的任务


我们再来看一个更简单的例子:

```python
def print_msg(msg) :
    """包裹层"""

    def printer() :
        """闭包"""
        print msg
    return printer
```

运行结果
```python
if __name__ == '__main__':
    temp = print_msg("hello world")
    temp()
```

`print_msg("hello world")`调用结束后, 传入的参数被保存在闭包中(`不过是否超出声明周期`), 如何验证这个说法呢

```python
#测试第一个例子, 显示是一个function(hello函数)
print temp.__closure__
(<cell at 0x10a449910: function object at 0x10a434ed8>,)

#测试第二个例子, 显示一个str(python2.7.8中的字符串类型, "hello world")
print temp.__closure__
(<cell at 0x1075b2948: str object at 0x1075bc060>,)
```

闭包装它所引用的自由变量添加到了`__closure`中


在python中创建一个闭包的过程:

1. 有一个外部函数A(工厂函数)
2. 外部函数A包裹一个内部函数B(闭包)
3. 内部函数B使用外部函数传入的自由参数(参数)
4. 外部函数A返回内部函数B

> 简而言之, 一个工厂函数创建并返回一个闭包, 并能够延长传入参数的生命周期(这里原理和`引用计数`有关, 不做深究)

#2. 装饰器

> 装饰器在python中`@`表示, 装饰器接受一个函数作为参数(或者更多参数), 返回一个函数


我们直接修改第一个例子

```python
# -*- coding: utf-8 -*-
#!/usr/bin/env python

from datetime import datetime

def print_now(fn) :
    def fn_wrap(*args, **kwargs) :
        print "now time: %s, the fn name: %s" % (datetime.now(), fn.__name__)
        return fn(*args, **kwargs)
    return fn_wrap

@print_now
def hello(name = "andrew") :
    print "Hello World! %s" % name

if __name__ == '__main__':
    hello()
```

先看一下输出结果
```python
now time: 2014-12-30 22:20:34.724911, the fn name: hello
Hello World! andrew
```

这中间发生了什么呢, 

```python
@print_now
def hello(name = "andrew") :
    print "Hello World! %s" % name
1. 当@print_now接一行接着一个函数, 此处相当于将hello函数作为参数传给print_now函数
2. 这三行代码相当于执行了hello = print_now(hello), 即将print_now返回的闭包再次赋值给hello, 相当于此时的hello已经指向了闭包, 还不在是原来的函数
3. 我们hello(), 就是调用闭包
```


再来看一个相对简单的例子:

```python
# -*- coding: utf-8 -*-
#!/usr/bin/env python

def print_msg(fn) :
    """包裹层"""

    def printer() :
        """闭包"""
        print "被传入的函数名称: %s" % fn.__name__
        return fn()
    return printer

@print_msg
def hello(name = "andrew") :
    print "Hello World! %s" % name

if __name__ == '__main__':
    hello()
    
#相当于
hello = print_msg(hello)
hello()
```

你是否能够分析出最终的输出结果呢?

```python
被传入的函数名称: hello
Hello World! andrew
```

#3. 何时使用闭包和装饰器

- 定义私有类属性

将`property`与装饰器结合实现属性私有化

```python
#python内建函数
property(fget=None, fset=None, fdel=None, doc=None)
```

`fget`是获取属性的值的函数,`fset`是设置属性值的函数,`fdel`是删除属性的函数,`doc`是一个字符串(like a comment).从实现来看，这些参数都是可选的

property有三个方法`getter()`, `setter()`和d`elete()` 来指定fget, fset和fdel。 这表示以下这行

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

- 对函数作简单的封装, 而不需要写一个类

如果只需要对函数作封装可以使用闭包, 而不需要专门创建一个类

```
def translator(frm = '', to = '', delete = '', keep = None) :
    if len(to) == 1 :
        to = to * len(frm)
    trans = string.maketrans(frm, to)
    if keep is not None :
        allchars = string.maketrans('', '')
        delete = allchars.translate(allchars, keep.translate(allchars, delete))
    def translate(s) :
        return s.translate(trans, delete)
    return translate


if __name__ == '__main__':
    digits_only = translator(keep = string.digits)
    print digits_only('Andrew Liu : 152-XXXX-XXXX')
    
#输出结果
152
```

- 惰性求值
- 提前赋值

#4. 更多阅读

[Python闭包的高级应用-装饰器的实现](http://www.cnblogs.com/inevermore/p/4189225.html)
[Python中的闭包](http://www.the5fire.com/closure-in-python.html)
[装饰器](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386819879946007bbf6ad052463ab18034f0254bf355000)
[Closures in Python](http://stackoverflow.com/questions/4020419/closures-in-python)

