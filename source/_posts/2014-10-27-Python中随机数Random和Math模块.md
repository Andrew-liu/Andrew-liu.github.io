---
layout : page
date: 2014-10-27 16:35:11 +0800
title: Python中随机数Random和Math模块
tags: Python
image:
  feature: abstract-6.jpg
commets: true
share: true
---

由于最近经常使用到Python中`random`,`math`和`time``datetime`模块, 所以决定花时间系统的学习一下

#1. math模块

> math中的函数不可以用于太过复杂的数的运算, 如果需要复杂数的运行最好使用`cmath`模块中同名函数, 如果想要更加高级的数学功能，可以考虑选择标准库之外的`numpy`和`scipy`模块，它们不但支持数组和矩阵运算，还有丰富的数学和物理方程可供使用

#1.1. 数学常量
- `math.pi` 这个数学常量等于 `3.141592...`
- `math.e` 这个数学常量 `e = 2.718281..., `

<!--more-->

#1.2. 常用简单函数
- `math.ceil(x)` : 对x向上取整，返回最小整数值大于或者等于x

```
# -*- coding:utf-8 -*-
import math  #仅在第一次声明, 以下都将省略
print math.ceil(math.pi)  #math.pi是圆周率pi, 类似于C/C++中的宏

//输出4
```

- `math.floor(x)` : 对x向下取整, 返回整数值小于或者等于x

```
>>> import math
>>> math.floor(math.pi)
3.0
```

- `math.pow(x,y)` : 指数运算，得到x的y次方

```
>>> math.pow(2, 3)
8.0
```

- `math.log(x[, base])` : 对数运算，默认基底为e的对数运算。使用base参数时，改变对数的基底, 变为以base为底的对数运算

```
>>> math.log(10)
2.302585092994046
>>> math.log(8, 2)   #log(x)/log(base).
3.0
```
- `math.sqrt(x) `    平方根计算

```
>>> math.sqrt(4)
2.0
```

- `math.fabs(x)`  取绝对值
- `math.factorial(x)` 求阶乘, 即x!
- `math.exp(x)` 求e的x次方


##1.3. 三角函数

> 以下函数都接收一个弧度(radian)为单位的x作为参数

```
math.acos(x) #求arccos(x)
math.asin(x) #求arcsin(x)
math.atan(x) #求arctan(x)
math.cos(x)  #求cos(x)
math.sin(x)  #求sin(x)
math.tan(x)  #求tan(x)
```

- `math.degrees(x)` 角度制转化为弧度制 
- `math.radians(x)` 弧度制转化为角度制

```
>>> math.degrees(math.pi / 2)
90.0
```


##1.5. 双曲函数和特殊函数
math.sinh(x), math.cosh(x), math.tanh(x), math.asinh(x), math.acosh(x), math.atanh(x)

> 还有些函数基本没用过



#2. random模块
> random模块的作用是产生随机数, 这个模块实现了伪随机数产生器


##1.1. 常用函数
- `random.seed([x])` 用户初始化一个随机数种子, 可选参数可以是任何hashtable对象,默认使用系统时间

- `random.randint(a, b)` 返回一个a到b之间的整数

```
>>> import random
>>> random.randint(3, 8)
7
```
- `random.randrange([start], stop[, step])` 从指定范围内，按指定基数递增的集合中 获取一个随机数。如：random.randrange(10, 100, 2)，结果相当于从[10, 12, 14, 16, ... 96, 98]序列中获取一个随机数。random.randrange(10, 100, 2)在结果上与 random.choice(range(10, 100, 2) 等效。

> random.randrange(start, stop, step)等价于random.choice(range(start, stop, step))

```
>>> random.randrange(10, 100, 2)
90
```

##1.2. 随机挑选和排序

- `random.choice(sequence)` : 从序列中获取一个随机元素. 参数sequence表示一个有序类型。这里要说明 一下：sequence在python不是一种特定的类型，而是泛指一系列的类型。list, tuple, 字符串都属于sequence

```
>>> random.choice(range(10))
1
>>> random.choice((1, 2, 3, 4))
3
```

- `random.sample(sequence, k)` # 从指定序列中随机获取指定长度k的片断。sample函数不会修改原有序列

```
>>> lst = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] 
>>> new_lst = random.sample(lst, 6)
>>> print new_lst
[8, 9, 2, 1, 5, 4]
>>> print lst
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

- `random.shuffle(x[, random])`，用于将一个列表中的元素打乱, 不会生成新的列表

```
>>> lst = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
>>> random.shuffle(lst)
>>> print lst
[10, 5, 2, 7, 3, 9, 4, 8, 6, 1]
```

##1.3. 随机生成实数

> 生成的实数符合均匀分布(`uniform distribution`)

- random.random()    随机生成下一个实数，它在[0,1)范围内。
- random.uniform(a,b) 随机生成下一个实数，它在[a,b]范围内。

```
>>> random.random()
0.019433835195078797
>>> random.uniform(3, 8)
6.830376841208885
```

- `random.gauss(mu,sigma)`    随机生成符合高斯分布的随机数，mu,sigma为高斯分布的两个参数。 
- `random.expovariate(lambd)`  随机生成符合指数分布的随机数，lambd为指数分布的参数。


> 其余是一些目前没用过的函数, 以后用到了再补充



#3. 参考链接

[random官网文档](https://docs.python.org/2/library/random.html)
[math官方文档](https://docs.python.org/2/library/math.html)
