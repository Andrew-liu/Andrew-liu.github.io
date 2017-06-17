title: "Python代码风格<译>"
date: 2014-11-27 22:33:55
tags: Python
---


---

> 其实翻译并没有想象中那么简单, 在豆瓣上看过很多对于翻译的吐槽, 没有真正去做一件事情就没有资格去评论其中的对与错, 简单的一次翻译体会到了翻译的个中艰辛, 这篇翻译主要起于自己想制定一个适合自己的Python编码规范, 其中内容并没有认真校对, 见谅, 能力有限. 有错误请指出, 感谢.



当你询问一个程序员为什么最爱Python语言, 他们常说它可读性很高. 确实, 高度的可读性是设计Python语言的核心.

一个使Python代码易读,易理解的原因是Python语言相对完善的编码风格和`Pythonic`的风格

并且, 当一个经验丰富的Python开发者指着一段代码说"这段代码不是Pythonic风格", 这常常意味着这些代码没有遵循一个普遍的代码准则, 没有通过最好的方式表达出它的意图.

在一些情况下, 没有一个约定的最好方式在Python代码中表达意图, 但这样的情况是比较少的.

<!--more-->

#1. 一般概念
---

##1.1. 代码明确
虽然任何类型的`黑魔法`都可以通过Python实现, 但是最明确和直接的方式是最好的

**Bad**

```python
def make_complex(*args):
    x, y = args
    return dict(**locals())
```

**Good**

```python
def make_complex(x, y):
    return {'x': x, 'y': y}
```

在上面优秀的代码中, 函数明确接收调用中x和y, 明确的返回字典. 开发者通过读第一行和最后一行可以知道函数的用法.

##1.2. 一行一句

虽然在一些复杂语句中被允许, 并展现这些语句的简洁和表达能力, 例如:列表生成式, 但将不相关的语句放在一行中是很差的代码风格

**Bad**

```python
print 'one'; print 'two'

if x == 1: print 'one'

if <complex comparison> and <other complex comparison>:
    # do something
```

**Good**

```python
print 'one'
print 'two'

if x == 1:
    print 'one'

cond1 = <complex comparison>
cond2 = <other complex comparison>
if cond1 and cond2:
    # do something
```

##1.3. 函数参数

函数的参数可以有四种不同的方式:

1. 位置参数

> 位置参数是任意且没有默认值的, 他们是最简单的参数形式, 可以被用少量的函数参数, 用来表达函数意义并且没有参数顺序. 例如, `send(message, recipient)`或者`point(x, y)`.

2. 关键字参数

> 关键字参数有默认值, 常用语函数参数可以可选. 当一个有多于两个或者三个位置函数, 函数样式就难以记忆, 这样使用有默认值的关键字参数是非常有帮助的. 例如, `send(message, to, cc = None, bcc = None)`. 这里`cc和bcc`是可选的, 当没有传值时默认为None

3. 任意参数列表

> 任意参数列表是函数传值的第三种方式, 如果一个函数意图通过定义可扩展数目的位置参数实现更好的表达, 那么可以定义`*args`参数. 在函数体中, `args`是所有剩余参数构成的元组. 例如,  `send(message, *args)`可以通过接受一下形式调用: `send('Hello', 'God', 'Mom', 'Cthulhu')`, 并且在函数体中 `args` 等于 `('God', 'Mom', 'Cthulhu')`.(就是将除了确定的参数, 剩余参数存储在一个元组中)


4. 关键字参数字典

> 关键字参数字典是最后一种函数传参, 如果一个函数需要不确定数目的参数, 可以使用`**kwargs`. 在函数体中`kwargs`是所有没有在函数定义中定义的参数形成的字典,

由程序员决定在写函数时使用哪一种参数. 如果上面的建议被采纳, 写出

- 易读的(名字和参数不需要解释)
- 已修改的(增加新的关键字参数不会影响其他部分的代码)

的参数是可能,也是享受的.

---

##1.4. 避免魔杖

`Hackers`, 强大的工具, Python提供丰富的`hooks`和`tools`, 允许做几乎所有令人棘手的技巧. 例如:

- 改变对象的创建和初始化
- 改变Python解释器引入模块
- 在Python中嵌入C语言

然后, 这些操作都有缺陷, 更好的方式是使用最直接的方式达到目的. 主要的缺陷在于当使用这些方式时, 可读性发生很大改变. 许多代码分析工具, 例如: `pylint`或者`pyflakes`不能分析这些`Magic`代码

Python开发者应该知道这些近乎无限的可能性, 因为使用这些方式没有不可解决的问题. 但是知道什么时候和怎么样使用他们是非常重要的.

##1.5. We are all consenting adults

通过上面知道Python有许多技巧, 其中一些是有潜在危险的.一个好的范例是任何用户都能重写对象的属性和方法: Python中没有`private`关键字.这种哲学不同于高等编程语言, 如Java, Java中又许多防止滥用的机制.这种哲学在表达`We are all consenting adults`

Python社区喜欢依赖一系列约定说明, 这些元素是不能直接访问的

私有属性和实现细节的主要约定是对所有`Internals`添加前缀下划线,如果用户破坏约定去访问标记元素, 当被修改的代码与客户代码有关, 可能产生任何问题

使用这些约定是被广泛鼓励的: 任何不想被客户代码使用的属性和方法都应该加上下划线前缀, 这样更易区分责任和修改代码;私有属性公有化总是有可能的, 而公有属性私有化可能更难操作.

##1.6. 返回值

当定义负责函数时, 常在函数体中使用多个返回语句. 然而, 为了保证明确的意图和可读性, 应在函数体中避免使用多个返回语句,

在函数中有两种返回值情况: 当函数正常处理,返回结果; 当一个错误情况产生, 支出错误输入参数或者其他原因导致函数不能完成任务.

如果在第二种情况不想抛出异常, 应该返回一个例如:`None或者False`的结果,指明函数没有正常执行. 在这种情况下, 最好返回检测到的错误信息. 这会帮助改善函数结构.

在正常情况下, 当一个函数中有多个停止点(`exit point`)时, 会变得很难去调式返回结果, 所以最好保持单个停止点. 这会帮助分离代码路径.

```python
def complex_function(a, b, c):
    if not a:
        return None  # Raising an exception might be better
    if not b:
        return None  # Raising an exception might be better
    # Some complex code trying to compute x from a, b and c
    # Resist temptation to return x if succeeded
    if not x:
        # Some Plan-B computation of x
    return x  # One single exit point for the returned value x will help
              # when maintaining the code.
```

#2. 惯用法(Idioms)
---

编程惯用法是一种代码编写的方式, Python惯用法通常被称为`Pythonic`.

下面是一些Python惯用法:

如果你知道list或者tuple的长度, 
1. 可以通过解包(unpacking)给它的元素命名.例如:`enumerate()`给list的每个元素提供两个元素的tuple

```python
for index, item in enumerate(some_list):
    # do something with index and item

```

2. 可以使用下面用法交换元素

```python
a, b = b, a
```

3. 嵌套解包任务

```python
a, (b, c) = 1, (2, 3)
```

4. 在Python3中, 引入了一个扩展解包的新方法

```python
a, *rest = [1, 2, 3]
# a = 1, rest = [2, 3]
a, *middle, c = [1, 2, 3, 4]
# a = 1, middle = [2, 3], c = 4
```

5. 创建一个可忽略(ignored)变量

如果需要赋值但不需要变量, 可以使用`__` (英文标点下的双下划线)


```python
filename = 'foobar.txt'
basename, __, ext = filename.rpartition('.')
```

> Note:
>> 许多Python风格建议使用`_`(单下划线)而不是双下划线. 单下划线普遍在`gettext()`函数中用作别名, 也常被用在互动操作中保存最后一次操作的值. 使用双下划线清晰方便, 并能够消除一些使用情况的干扰

6. 创建一个长度为N的list, 其中所有的元素相同

```python
four_nones = [None] * 4
```

7. 创建一个长度为N的list

因为list是可变的, *操作对于同样的list创建N引用的list,不会像我们想象中那样(有点疑惑).正确写法应该使用列表生成式

```python
four_lists = [[] for __ in xrange(4)]
```

8. 对`空字符串`使用`str.join()`来创建一个字符串是一个普遍用法

```python
letters = ['s', 'p', 'a', 'm']
word = ''.join(letters)

#这种用法可以用在list和tuple
```
有时我们需要在很多东西中搜索, 看一下:list和dictionary

```python
d = {'s': [], 'p': [], 'a': [], 'm': []}
l = ['s', 'p', 'a', 'm']

def lookup_dict(d):
    return 's' in d

def lookup_list(l):
    return 's' in l
```

虽然函数看起来一样, `lookup_dict`利用哈希表的特性,两者性能完全不同,在list需要遍历每个元素查找,非常耗时. 通过分析字典的哈希, 找到字典中的key非常快速.

##2.1. Zen of Python

也被称称为[PEP 20](https://www.python.org/dev/peps/pep-0020), Python设计准则

```python
>>> import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

优秀的python的风格范例, 可以看 [this Stack Overflow question](http://stackoverflow.com/questions/228181/the-zen-of-python) 或者 [these slides from a Python user group](http://artifex.org/~hblanks/talks/2011/pep20_by_example.pdf)

##2.2. PEP8

[PEP8](https://www.python.org/dev/peps/pep-0008)是Python编码风格.

使你的代码遵照PEP8是一个很好的主意, 当团队项目开发时, 使代码一致,[PEP8 命令行项目](https://github.com/jcrocholl/pep8)帮助你检查代码是否符合PEP8,安装这个工具在命令行运行下面的命令:

    $ pip install pep8

然后在文件或者很多文件中运行,会得到违反PEP8的报告

```python
$ pep8 optparse.py
optparse.py:69:11: E401 multiple imports on one line
optparse.py:77:1: E302 expected 2 blank lines, found 1
optparse.py:88:5: E301 expected 1 blank line, found 0
optparse.py:222:34: W602 deprecated form of raising exception
optparse.py:347:31: E211 whitespace before '('
optparse.py:357:17: E201 whitespace after '{'
optparse.py:472:29: E221 multiple spaces before operator
optparse.py:544:21: W601 .has_key() is deprecated, use 'in'
```

#3. 约定
---

遵循下面的约定会使你的代码可读性更高.

1. 检查是否变量等于一个常量

不需要明确的与`None, True, 0`比较, 只需要直接使用`if语句`, 那些值被认为是`false`可以查看[Truth Value Testing](https://docs.python.org/2/library/stdtypes.html#truth-value-testing)

**Bad**

```python
if attr == True:
    print 'True!'

if attr == None:
    print 'attr is None!'
```

**Good**

```python
# Just check the value
if attr:
    print 'attr is truthy!'

# or check for the opposite
if not attr:
    print 'attr is falsey!'

# or, since None is considered false, explicitly check for it
if attr is None:
    print 'attr is None!'
```

2. 访问字典元素

不要使用`dict.has_key()`方法, 使用`x in d`语法或者传入一个默认参数到`dict.get()`

**Bad**

```python
d = {'hello': 'world'}
if d.has_key('hello'):
    print d['hello']    # prints 'world'
else:
    print 'default_value'
```

**Good**

```python
d = {'hello': 'world'}

print d.get('hello', 'default_value') # prints 'world'
print d.get('thingy', 'default_value') # prints 'default_value'

# Or:
if 'hello' in d:
    print d['hello']
```

3. 列表操作使用简洁方式

[列表生成式](https://docs.python.org/2/tutorial/datastructures.html#list-comprehensions)提供强大简洁的list工作方式.`map()和filter()`函数使用不同的, 更简洁的语法操作list

**Bad**

```python
# Filter elements greater than 4
a = [3, 4, 5]
b = []
for i in a:
    if i > 4:
        b.append(i)
```

**Good**

```python
a = [3, 4, 5]
b = [i for i in a if i > 4]
# Or:
b = filter(lambda x: x > 4, a)
```

**Bad**

```python
# Add three to all list members.
a = [3, 4, 5]
for i in range(len(a)):
    a[i] += 3
```

**Good**

```python
a = [3, 4, 5]
a = [i + 3 for i in a]
# Or:
a = map(lambda i: i + 3, a)
```

使用`enumerate()`保持list计数

```python
a = [3, 4, 5]
for i, item in enumerate(a):
    print i, item
# prints
# 0 3
# 1 4
# 2 5
```

4. 读取文件

使用`with open`语法读取文件, 会自动关闭读取文件

**Bad**

```python
f = open('file.txt')
a = f.read()
print a
f.close()
```

**Good**

```python
with open('file.txt') as f:
    for line in f:
        print line
```

`with`语句更好是因为它总是会去确定你的文件是否关闭, 即使`with`中引发异常

5. 连接行

当一个逻辑行长度超过可接受的限制, 需要划分为多行, 如果一行最后一个字符是`反斜线\`,Python解释器会连接下一行.这在一些情况下很有帮助, 但是它`非常脆弱`应该避免使用: 在行末反斜线加一个空格,会破坏代码并可能产生意外结果.

一个更好的解决方案是使用圆括号括住代码. 没有匹配的圆括号.使用中括号和花括号也能起到效果.

**Bad**

```python
my_very_big_string = """For a long time I used to go to bed early. Sometimes, \
    when I had put out my candle, my eyes would close so quickly that I had not even \
    time to say “I’m going to sleep.”"""

from some.deep.module.inside.a.module import a_nice_function, another_nice_function, \
    yet_another_nice_function
```
**Good**

```python
my_very_big_string = (
    "For a long time I used to go to bed early. Sometimes, "
    "when I had put out my candle, my eyes would close so quickly "
    "that I had not even time to say “I’m going to sleep.”"
)

from some.deep.module.inside.a.module import (
    a_nice_function, another_nice_function, yet_another_nice_function)
```

然后, 更多时候对于很长的逻辑行并没有划分, 你试图在一行做更多的事情, 这会妨碍可读性.

翻译结束...


[Code Style](http://docs.python-guide.org/en/latest/writing/style/#code-style)
