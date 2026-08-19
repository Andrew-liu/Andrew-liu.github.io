title: Python编码规范
date: 2014-11-28 09:24:08
tags: Python
---


> `Python`,一门简洁,全能的胶水语言, 牺牲性能换来的`多面手`的特性, 最近写Python比较多, 通过这篇博文我试图形成自己的编码规范.为以后的代码重构和阅读埋下铺垫. 主要参考的是google编码规范, 毕竟大公司


> 大部分.py文件不必以#!作为文件的开始. 程序的main文件以 #!/usr/bin/python2或者 #!/usr/bin/python3开始.(`#!先用于帮助内核找到Python解释器, 但是在导入模块时, 将会被忽略. 因此只有被直接执行的文件中才有必要加入#!.`)




##1.1. 分号

> 不要在行尾加分号, 也不要用分号将两条命令放在同一行

可能之前用C/C++比较多, 刚开始写Python的时候, 总是会在行尾加上分号,而且不会报错, 不过最近python写的比较多,所以也改过来了

<!--more-->


##1.2. 行长度

> 每行不超过80个字符

例外:

- 常的导入模块语句
- 注释里面的URL

使用`圆括号`将行隐式连接起来(`Python会将 圆括号, 中括号和花括号中的行隐式的连接起来`)

```py
my_very_big_string = (
    "For a long time I used to go to bed early. Sometimes, "
    "when I had put out my candle, my eyes would close so quickly "
    "that I had not even time to say “I’m going to sleep.”"
)
```

##1.3. 括号

> 除了行隐式, 不要在返回语句和条件语句中使用括号, 可以在元组两边使用括号

```py
if foo:
    bar()
while x:
    x = bar()
if x and y:
    bar()
if not x:
    bar()
return foo
```

##1.4. 缩进

> 使用四个空格缩进(习惯使用Tab缩进, 貌似这样缺陷很大, 以后改正)

##1.5. 空行

- 顶级定义之间空两行(比如函数定义之间或者类定义之间)
- 方法定义之间空一行(类定义与第一个方法空一行`这个我以前也是这么做的`)
- 按照标准排版规范使用标点两边空格
    - 不要在逗号, 分号, 冒号前面加空格, 但应该在它们后面加
    - 参数列表, 索引或切片的左括号前不应加空格
    - 在二元操作符两边都加上一个空格, 比如`赋值(=), 比较(==, <, >, !=, <>, <=, >=, in, not in, is, is not), 布尔(and, or, not)`.
    - 当’=’用于指示关键字参数或默认参数值时, 不要在其两侧使用空格
    - 不要用空格来垂直对齐多行间的标记, 因为这会成为维护的负担(`注意这点`)

##1.6. 注释

- 文档字符串的惯例是使用三重双引号"""(`一个文档字符串应该这样组织: 首先是一行以句号,
问号或惊叹号结尾的概述(或者该文档字符串单纯只有一行). 
接着是一个空行. 接着是文档字符串剩下的部分, 
它应该与文档字符串的第一行的第一个引号对齐`)

- 函数和方法(`一个函数必须要有文档字符串, 除非外部不可见, 非常短小, 简单明了, 
文档字符串应该包含函数做什么, 以及输入和输出的详细描述.对于复杂的代码, 
在代码旁边加注释会比使用文档字符串更有意义, 注释应该至少离开代码2个空格`)

Args:
    
    列出每个参数的名字, 并在名字后使用一个冒号和一个空格, 分隔对该参数的描述.如果描述太长超过了单行80字符,使用2或者4个空格的悬挂缩进(与文件其他部分保持一致). 描述应该包括所需的类型和含义. 如果一个函数接受*foo(可变长度参数列表)或者**bar (任意关键字参数), 应该详细列出*foo和**bar.

Returns: (或者 Yields: 用于生成器)
    
    描述返回值的类型和语义. 如果函数返回None, 这一部分可以省略.

Raises:
    
    列出与接口有关的所有异常.
    
```py
def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
    """Fetches rows from a Bigtable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by big_table.  Silly things may happen if
    other_silly_variable is not None.

    Args:
        big_table: An open Bigtable Table instance.
        keys: A sequence of strings representing the key of each table row
            to fetch.
        other_silly_variable: Another optional variable, that has a much
            longer name than the other args, and which does nothing.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {'Serak': ('Rigel VII', 'Preparer'),
         'Zim': ('Irk', 'Invader'),
         'Lrrr': ('Omicron Persei 8', 'Emperor')}

        If a key from the keys argument is missing from the dictionary,
        then that row was not found in the table.

    Raises:
        IOError: An error occurred accessing the bigtable.Table object.
    """
    pass
```

- 类

类应该在其定义下有一个用于描述该类的文档字符串. 如果你的类有公共属性(Attributes), 那么文档中应该有一个属性(Attributes)段. 并且应该遵守和函数参数相同的格式.
    
```py
class SampleClass(object):
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """

    def __init__(self, likes_spam=False):
        """Inits SampleClass with blah."""
        self.likes_spam = likes_spam
        self.eggs = 0

    def public_method(self):
        """Performs operation blah."""
```



##2.1. 类

> 如果一个类不继承自其它类, 就显式的从`object`继承. 嵌套类也一样

继承自 object 是为了使属性(properties)正常工作, 并且这样可以保护你的代码, 使其不受`Python 3000`的一个特殊的潜在不兼容性影响. 这样做也定义了一些特殊的方法, 这些方法实现了对象的默认语义, 包括 `__new__, __init__, __delattr__, __getattribute__, __setattr__, __hash__, __repr__, and __str__`


##2.2. 字符串

> 即使参数都是字符串, 使用%操作符或者格式化方法格式化字符串

- 避免在循环中用`+和+=`操作符来累加字符串. 

> 字符串是不可变的

这样做会创建不必要的临时对象, 并且导致二次方而不是线性的运行时间. 作为替代方案, 你可以将每个子串加入列表, 然后在循环结束后用` .join` 连接列表.

- 在同一个文件中, 保持使用字符串引号的一致性(只用双引号或者只用单引号)
- 为多行字符串使用三重双引号`"""`

##3.1. 文件和sockets

> 在文件和sockets结束时, 显式的关闭它.(文件使用`with`)

##3.2. TODO注释

> 为临时代码使用TODO注释

TODO注释应该在所有开头处包含”TODO”字符串, 紧跟着是用括号括起来的你的名字, email地址或其它标识符. 然后是一个可选的冒号. 接着必须有一行注释, 解释要做什么

```py
# TODO(kl@gmail.com): Use a "*" here for string repetition.
# TODO(Zeke) Change this to use relations.
```

##3.3. import格式

> 每个导入应该独占一行

导入总应该放在文件顶部, 位于模块注释和文档字符串之后, 模块全局变量和常量之前. 导入应该按照从最通用到最不通用的顺序分组,
每种分组中, 应该根据每个模块的完整包路径按字典序排序, 忽略大小写:

1. 标准库导入
2. 第三方库导入
3. 应用程序指定导入

```py
import foo
from foo import bar
from foo.bar import baz
from foo.bar import Quux
from Foob import ar
```


##3.4. 语句

> 每个语句独占一行

##3.5. 访问控制

在Python中, 对于琐碎又不太重要的访问函数, 你应该直接使用公有变量来取代它们, 这样可以避免额外的函数调用开销. 当添加更多功能时, 你可以用属性(property)来保持语法的一致性.

另一方面, 如果访问更复杂, 或者变量的访问开销很显著, 那么你应该使用像 `get_foo()` 和 `set_foo()` 这样的函数调用. 如果之前的代码行为允许通过属性(property)访问 , 那么就不要将新的访问函数与属性绑定. 这样, 任何试图通过老方法访问变量的代码就没法运行, 使用者也就会意识到复杂性发生了变化.


##3.6.命名

> `module_name, package_name, ClassName, method_name, ExceptionName, function_name, GLOBAL_VAR_NAME, instance_var_name, function_parameter_name, local_var_name.`


1. 避免使用单字符名称, 除了计数器和迭代器
2. 避免使用包/模块名中连字符(`-`)
3. 避免使用双下划线开头和结尾的名称('Python保留')

命名约定:

- 用单下划线开头(`_`)表示模块变量或者函数是`protected`(使用import * from不会包含)
- 双下划线(`__`)表示实例变量或者方法类内私有
- 将相关的类和顶级函数放在一个模块里
- 对类名使用大写字母开头的单词(如`RandomForest`).模块名小写加下划线(`lower_with_under.py`),

##3.7. Main

应该在执行主程序前总是检查 `if __name__ == '__main__'` 

```py
def main():
      ...

if __name__ == '__main__':
    main()
```

[What does `if __name__ == “__main__”:` do?
](https://stackoverflow.com/questions/419163/what-does-if-name-main-do)
