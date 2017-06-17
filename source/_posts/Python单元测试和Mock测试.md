title: Python单元测试和Mock测试
date: 2015-12-12 00:34:16
tags: Python
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

## 单元测试

- 测试可以保证你的代码在一系列给定条件下正常工作
- 测试允许人们确保对代码的改动不会破坏现有的功能
- 测试迫使人们在不寻常条件的情况下思考代码，这可能会揭示出逻辑错误
- 良好的测试要求模块化，解耦代码，这是一个良好的系统设计的标志


<!--more-->

## 范例

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os, sys
import time, datetime
import unittest
from unittest import TestCase

class TestSequenece(TestCase):

    def setUp(self):
        self.lst = range(10)
        print "setUp starting ..."

    def test_eq(self):
        print "test_eq starting..."
        self.assertEqual(self.lst, range(10))

    def test_in(self):
        print "test_in starting..."
        self.assertIn(1, self.lst)
        self.assertNotIn(10, self.lst)

    def test_instance(self):
        print "test_instance starting..."
        self.assertIsInstance(self.lst, list)

    def tearDown(self):
        print "tearDown starting..."

if __name__ == '__main__':
    unittest.main()
```

然后我们看一下执行结果再分析:

```
setUp starting ...
test_eq starting...
tearDown starting...
.setUp starting ...
test_in starting...
tearDown starting...
.setUp starting ...
test_instance starting...
tearDown starting...
.
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```

**共运行三个测试, 每次测试成功通过都会输出一个.号**

- `TestCase`直译就是测试用例, 一个测试用例可以包含多个测试
- `test_xxxx`就是测试项, 根据实际的功能代码逻辑来编写对应的测试项, 运行时会自动查找所有以test开发的成员函数
- `assertXXXX`断言语句, 用来判断测试结果是否符合测试预期结果.
- `setUp`是执行`每个`测试项前的准备工作, 比如：可以做一些初始化工作
- `tearDown`是执行在`每个`测试项后的收尾工作,销毁测试过程中产生的垃圾, 恢复现场等

## Mock测试


Mock测试是什么鬼? 我们常常遇到这样一种场景, 我们测试一些函数, 而这些函数内部调用另外带有`副作用`的操作, 这可能导致我们在测试过程中对数据造成未知的副作用, 而这并不是我们希望在测试中看到的.

Mock测试可以替换到指定的Python对象或者方法, 并自定义指定对象或者方法的返回值, 从来模拟对象或者方法, 消除`副作用`.

> Mock在Python3.3时加入到标准库中, 2.X版本可以通过pip安装

```
$ pip install mock
```

首先任意写一个函数

```
# -*- coding: utf-8 -*-
#!/usr/bin/env python

import os, sys, time

def foo():
    lst = [1]
    lst = give_me_five(lst)
    return lst

def give_me_five(lst):
    return lst * 5
```

我们希望通过单元测试来测试这个函数的逻辑正确性(`请注意, 这只是一个演习!`).

```
# -*- coding: utf-8 -*-
#!/usr/bin/env python

import os, sys, time
#  sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
import unittest
from unittest import TestCase
import mock
import module


class Foo(object):
    pass

class TestMock(TestCase):
    # 1
    def test_method(self):
        obj = Foo()
        obj.method = mock.MagicMock(return_value=3)
        print obj.method
        self.assertEqual(obj.method(4), 3)
    # 2
    @mock.patch('module.foo')
    def test_decorator(self, foo):
        #  res = module.foo()
        foo.return_value = [1, 2, 3]
        self.assertEqual(foo(), [1, 2, 3])
    # 3
    def test_with(self):
        with mock.patch('module.give_me_five') as give_me_five:
            give_me_five.return_value = "I'm Mock"
            self.assertEqual(module.foo(), "I'm Mock")
    # 4
    def test_module(self):
        module.give_me_five = mock.Mock(return_value=[1] * 5)
        module.give_me_five([1])  # 此时已经变成了一个Mock对象, 并尝试调用
        module.give_me_five.assert_called_with([1])  # 对mock的参数进行断言
        self.assertEqual(module.foo(), [1] * 5)

if __name__ == '__main__':
    unittest.main()
```

- 我们首先集成TestCase创建了一个单元测试
- `# 1`位置, 我们通过mock提供的函数给obj的method方法设置返回值(可以看到类中并不包含method方法). 最后通过断言来判断返回值等于我们通过`MagicMock`设置的返回值
- `# 2`位置, 我们通过mock提供的装饰器, `patch()可以作为函数做装饰, 类装饰器, 上下文管理器` 将module中的foo函数给mock掉, `并且并mock的函数生成的Mock对象作为类成员函数参数传入`,  指定了foo函数的返回值, 并通过了断言测试
- `# 3`位置, 将patch()作为一个上下文管理, 关于上下文管理器可以看我另一篇文章[Python奇技淫巧](http://andrewliu.in/2015/11/14/Python%E5%A5%87%E6%8A%80%E6%B7%AB%E5%B7%A7/#with的魔力), 用法和作为装饰器基本类似
- `# 4`位置, 我们调用`module.foo`函数, 而我们并不关系foo()调用了那些函数, 我只关心在成功调用`module.give_me_five`后, foo函数的逻辑正确性. 所以此次我们通过Mock函数给`module.give_me_five`指定我们希望的返回值. 这样就能独立的测试`module.foo`的逻辑


> mock的主要思想: 通过mock对象对某些函数进行替换, 对在测试上下文中, 这些被mock的函数被重定向到指定的mock对象

**mock还有一些更高级的应用**

- `MagicMock`是`Mock`的子类, 并且包含一些如`__str__`一样的黑魔法函数, 使用`MagicMock`甚至可以mock掉黑魔法函数
- 通过patch.object可以mock掉类中指定的成员函数
- 通过patch.dict可以将对象mock为字典
- 通过patch中的`start`和`stop`方法可以控制mock的生效范围, 更加灵活的运行mock测试

## 参考链接


- [Python unittest官方文档](https://docs.python.org/2/library/unittest.html)
- [Python中的单元测试](http://pm.readthedocs.org/zh_CN/latest/unittest/python.html)
- [Python 使用断言的最佳时机](http://www.oschina.net/translate/when-to-use-assert)
- [Python Mock的入门](http://segmentfault.com/a/1190000002965620)
- [The Mock Class](http://www.voidspace.org.uk/python/mock/mock.html)
