title: 快学Lua
date: 2016-08-29 16:01:26
tags: Lua
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

> 工作需要用到Lua做一些脚本, 所以学习一下这个在游戏开发应用广泛的语言, 当然, 我更喜欢称之为撸啊撸语言...


## 简介

- `Lua`是一种轻量语言，它的官方版本只包括一个精简的核心和最基本的库, 
- 性能方面, Lua比Python快(这也是脚本语言选型中不用python的原因)
- Lua与C/C++交互方便, 易于扩展
- `Lua`是一个动态弱类型语言, 支持增量式垃圾收集策略, 支持`协程`
- 支持REPL(Read-Eval-Print Loop, 交互式解释器)

```python
-- 命令行中输入lua, 使用REPL
andrew_liu:Lua/ $ lua  
Lua 5.2.4  Copyright (C) 1994-2015 Lua.org, PUC-Rio
> print "hello world"  -- 国际编程学习惯例
hello world
> print("hello world")
hello world
>
```

<!--more-->

## 安装

已经懒得再安利`brew`了(不好用你干我), 一行命令完成安装

```
# 安装路径为/usr/local/Cellar/lua
$ brew install lua
```

## 基础语法

### 注释

> 单行注释使用`--`, 多行注释使用`--[[`和`--]]`

```
-- 这种为单行注释
-- Date: 16/8/27
-- Time: 22:33

--[[
    这里是多行注释
--]]
```

### 变量定义

- lua中的所有变量默认为全局变量, 定义局部变量需要使用`local关键字`(是不是很变态)
- **所有的数字**都是双精度浮点型(double)
- 字符串可以使用单引号和双引号, 类似于`python`中的字符串表示形式
- 使用一个`未定义的变量`不会报错, 而是返回nil(类似于C/C++中的NULL)
- 一行语句可以同时定义多个变量

```
-- variable
age = 0 -- 全局变量
sum = 2  -- 所有数字都是double型
str = '蛤丝'  -- 字符串可以是单引号和双引号
str1 = "还是蛤丝"
-- 使用`[[]]`定义多行字符串时, 其空格都会被保留
str2 = [[多行的字符串
    以两个方括号
    开始和结尾。]]
sum = nil  -- 撤销sum的定义

test = not_define
print(test)
name, age, sex = "andrew", 18, "male", "蛤丝"  -- 最后一个变量会被丢弃, 而不会报错(闷死发大财才是最吼得)
```

### if-else

> if-else与python有些像, 不过需要注意if和elseif最后的`then`和末尾的`end`

```
-- if-else
age = 100
if age <= 40 and  age >= 20 then
    print("young man!")
elseif age > 40 and age < 100 then
    print("old man")
elseif age < 20 then
    print("too young")
else
    print("monster")
end
```

### 循环

- `Lua`同样支持三种循环, while循环, for循环, until循环(可以理解为C/C++中的do-while循环)

**while循环**

当条件满足, 会一直执行循环体
```
-- while
num = 0
while num < 10 do
    print(num)
    num = num + 1
end
```

**for循环**

```
-- for
sum = 0
--  1表示从1开始, 10表示最大为10(包含10)
-- C++: for (int i = 1, i <= 10; ++i)
for i = 1, 100 do  -- 包含1和100
    sum = sum + i
end
-- 类似于Python中的range操作, 2表示, 每次循环后i执行+2操作
-- C++: for (int i = 1, i <= 10; i = i + 2)
for i = 1, 10, 2 do  -- 1,3,5,7,9
    print(i)
    sum = sum + i
end
-- ..操作符用于连接字符串, 可以理解为python中字符串的+
操作
print("sum = " .. sum)
```

**until循环**, 类似于C/C++中的do-while语句

```
-- until(do-while)
sum = 2
-- 中文: 直到sum的值大于1000时, 才终止循环
repeat
    sum = sum ^ 2  -- 幂操作
until sum > 1000  
print("do-while, sum = "..sum)
```

### 函数

- 函数在Lua中是一等公民
- 支持有名和匿名函数
- 函数最后有个end, 写C/C++/Python的可能不太适应
- 函数支持一次`返回多个变量
- 局部函数的定义只需要在函数前加个`local`, 和局部变量类似

```
-- function(return支持返回多个值), 函数是一等公民
function add(a, b)
    return a + b
end
print("1 + 2 = "..add(1, 2))
-- 匿名函数
f = function (a, b)
    return a * b
end
print ("匿名函数: " .. f(2, 3))
```

**递归(经典: 斐波那契)**

```
function fib(n)
  if n < 2 then return n end
  return fib(n - 2) + fib(n - 1)
end
```


## Table

`Table`是Lua中核心数据结构, 我理解为其他语言中的`Dict/Map`数据结构(key-value的数据结构), 而数组等在Lua中都是通过`Table`来表示的

> Table可以使用任何非nil的值作为key

```
-- Lua唯一的组合数据结构
-- 可以使用任何非nil值作为key
-- Table(key-value数据结构)
map = {
    name = "andrew",  -- 此处name默认使用字符串作为key
    age = 18,
    sex = "male"
}
-- 上下两种写法等价
--map = {['name'] = 'andrew', ['age'] = 18, ['sex'] = 'male'}  
```

对Table创建, 读取, 更新和删除操作, key-value的读取可以通过`.`符号

```
-- Table的CRUD
map['hobby'] = "膜蛤"  -- map.hobby = "膜蛤"
print(map.name)
map.age = 19
map.sex = nil  -- 删除key
```
遍历一个Table

```
-- 遍历一个Table
for k, v in pairs(map) do
    print(k, v)
end
```

### 数组/列表

- 数组是通过Table来实现的

> `Lua`中数组的下标是从1开始, 从1开始, 从1开始. 重要的事情说三遍.

```
-- array, 数组下标从1开始
-- 数组中可以是不同类型的数据
arr = {"string", 100, "andrew", function() print("hello world") end }
-- 等价于
-- arr = {[1]="string", [2]=100, [3]="andrew", [4]=40, [5]=function() print("hello world") end}

for i = 1, #arr do  -- #arr表示数组的长度
    print(arr[i])
end
```


### 元表(metatable)

Table的`metatable`提供一种类似于操作符重载的机制, 写个python的可能重写个`__add__`方法, 感觉有点类似

- 通过`setmetatable`将自己实现的metamethod绑定到对应的Table对象中

```
-- table的元表提供一种机制,支持操作符重载
data_one = {a = 1, b = 2}  -- data_one和data_two 可以看做表示1/2和3/4
data_two = {a = 3, b = 4 }
meta_fraction = {}  -- 新定义一个元表
function meta_fraction.__add(f1, f2) -- __add是build-in的原方法
    local sum = {}
    sum.b = f1.b * f2.b
    sum.a = f1.a * f2.b + f2.a * f1.b
    return sum
end
setmetatable(data_one, meta_fraction)  -- 为之前定义的两个table设置metatable
setmetatable(data_two, meta_fraction)
s = data_one + data_two  -- 执行 + 操作
```

- 元表的`__index` 可以重载用于查找的点操作符, 我理解有点类似于python中的`__getattr__`, 对类中的属性做查找

```
-- 当表要索引一个值时如table[key], Lua会首先在table本身中查找key的值,
-- 如果没有并且这个table存在一个带有__index属性的Metatable, 则Lua会按照__index所定义的函数逻辑查找
default_map = {name = 'andrew', sex = 'male' }
new_map = {age = 18 }
setmetatable(default_map, {__index = new_map})
print(default_map.name, default_map.age)  -- 继承了new_map的属性
```

### 元方法(metamethod)

`__add`这种方法在Lua中被称为元方法, 

```
-- __add(a, b)                     for a + b 
-- __sub(a, b)                     for a - b 
-- __mul(a, b)                     for a * b 
-- __div(a, b)                     for a / b 
-- __mod(a, b)                     for a % b 
-- __pow(a, b)                     for a ^ b 
-- __unm(a)                        for -a 
-- __concat(a, b)                  for a .. b 
-- __len(a)                        for #a 
-- __eq(a, b)                      for a == b 
-- __lt(a, b)                      for a < b 
-- __le(a, b)                      for a <= b 
-- __index(a, b)  <fn or a table>  for a.b 
-- __newindex(a, b, c)             for a.b = c 
-- __call(a, ...)                  for a(...) 
```

### 继承

`继承`同样是通过元表和元方法来实现的.

```
-- table实现继承
Dog = {}
-- 这个new函数我理解为继承类的构造函数, 冒号(:)函数会使参数默认增加一个self作为参数
function Dog:new()
    -- 下面这句相当于继承的类增加的新的成员变量
    local new_obj = {sound = 'woof'}
    self.__index = self
    return setmetatable(new_obj, self)
end
function Dog:make_sound()  -- 等价于Dog.make_sound(Dog)
    print("say " .. self.sound)
end
-- 使用
new_dog = Dog:new()  -- new_dog获得Dog的方法和变量列表
new_dog:make_sound()
```


1. `self.__index = self`是防止self被扩展后改写，所以，让其保持原样
2. `setmetatable(new_obj, self)`会返回第一个参数, new_obj会得到self的函数
3. `Dog:new()`调用方式会增加一个隐式参数`self`

## 模块

1. require函数, 第一次执行后有缓存, 所以载入同样的lua文件时, 只有第一次的时候会去执行, 多次执行同一句require, 只有第一次生效
2. dofile函数, 类似于require, 但无缓存, 每次语句都会重新执行一次
3. loadfile加载一个lua文件, 但是并不运行它, 只要在需要的时候再像函数一样执行它


```
-- module
-- 1. require
local test = require('module')  -- require文件会被缓存
-- test.say_my_name()  -- 不能调用local
test.say_hello()
-- 2. dofile 类似require 但不缓存
-- 3. loadfile加载一个lua文件, 但是并不运行它
```

## 参考链接

- [lua程序设计](http://book.luaer.cn/)
- [Lua 5.1 参考手册(云风译)](http://www.codingnow.com/2000/download/lua_manual.html)
- [Learn Lua in Y Minutes](https://learnxinyminutes.com/docs/zh-cn/lua-cn/)
- [Lua简明教程](http://coolshell.cn/articles/10739.html)

