title: Lua编码规范
date: 2016-09-04 21:08:18
tags: Lua
---

> 写代码是为了给别人读的, 所以总需要一些建议性的规范, 所以简单的翻译了一下国外的这篇[Lua编码规范](http://lua-users.org/wiki/LuaStyleGuide), 并做了一些适合自己的改动.

## Introduction

代码写出来是给人读的, 保持`一致性`能够增加改善代码的可读性. `一致性(Consistency)`在项目之间, 项目内部, 单个模块或单个函数中非常重要. 值得注意的是, 编码规范只是`一种建议`, 有时可能并不需要去遵守编码规范.


<!--more-->

Lua包含自己的语法和常用方式, 本文的编码规范受到其他语言的编码规范的启发:

- [Python Style Guide](https://www.python.org/dev/peps/pep-0008/)
- [Perl Style Guide](http://search.cpan.org/dist/perl/pod/perlstyle.pod)
- [Various Primarily C/C++ Style Guides](https://google.github.io/styleguide/cppguide.html)

> **编码规范是一种艺术**. 

在定义`Lua`的风格建议时, 从一些`Lua`的源码中获得灵感(这些源码来自官方文档, Lua的作者们, 其他有名的源码) :

- [Programming in Lua](http://www.lua.org/pil/) by Roberto Ierusalimschy
- [Lua 5.1 Reference Manual](http://www.lua.org/manual/5.1/) by R. Ierusalimschy, L. H. de Figueiredo, W. Celes
- Other [LuaBooks](http://lua-users.org/wiki/LuaBookshttp://www.keplerproject.org/) such as Beginning Lua Programming
- Examples Lua programs in the "test" folder of the Lua distribution.
- Well known modules such as in [Kepler](http://www.keplerproject.org/) or on [LuaForge](http://lua-users.org/wiki/LuaForge).


## Formatting

**缩进(Indentation)** 缩进常常使用`四个空格缩进`(原文建议为两个空格). 使用四个空格缩进原因在于, 公司中C/C++编码风格和Python编码风格都是遵循四个缩进的.

```
t = {4, 5, 6, "hello"}
for i, v in ipairs(t) do
    if type(v) == "string" then
        print(v)
    else
        print("index = " .. i .. ", num = " .. v)
    end
end
```

## Naming

**变量命名长度(Variable name length)**: 这个一个一般的规则, 范围大的变量名应该比范围小的变量有更多的描述词. 比如, 在一个大型程序中`i`作为全局变量的命名是非常糟糕的, 但作为一个小范围的计数器是最吼得.

**值和对象命名(Value and object variable naming)**: 保存值和对象的变量的命名应该一般是`小写`并且比较短.

- 布尔变量(Booleans): 布尔值或者函数的前缀应该对整体的意义是有帮助的. 例如`is_directory`而不是`directory`

**函数命名(Function naming)**: 函数的命令规则一般类似于值和对象命名. 应该使用下划线和小写单词组合(原文使用getmetatable这种多个字母写在一起的方式). 例如`print_table`

**Lua内部变量命名(Lua internal variable naming)**: 使用名字以下滑线开头, 后跟大写字母作为传统, 如(`_VERSION`). 这种命名常用于`常量`, 但不是必须的.

**常量命名(Constants naming)**: 常量, 尤其是值比较简单的, 使用全大写的组合(`ALL_CAPS`).

**模块/包命名(Mudule/Package naming)**: 使用小写字母加下划线组合的形式, 如: `lua_module.lua`

> 变量名只有一个`_`的变量, 表示希望忽略这个变量, 这只是一个`传统(convention)`

```
-- 忽略数组下标
for _, v in ipairs(t) do print(v) end
```

`i, k, v, t`常用于一下场景

```
for k,v in pairs(t) ... end
for i,v in ipairs(t) ... end
mt.__newindex = function(t, k, v) ... end
```

`M`常用于表示`当前模块的table(current module table)`

**类名(Class names)**: 以大驼峰形式命名(`ClassName`).

**匈牙利命名法?(Hungarian notations, 是这么翻译吗)**. ~~将语义信息编码到变量中能够帮助增加代码可读性, 尤其是当编码的含义不容易被理解时, 但是过度的命名可能会显示冗余并且降低复杂度(在黑OC呢?). 在一些静态语言中(如, C), 变量类型已知, 此时将变量类型加入命名是很冗余的.~~

```
local function paint(canvas, n_times)
    for i = 1, n_times do
        local hello_str_asc_lc_en_const = "hello world"
        canvas:draw(hello_str_asc_lc_en_const:to_upper())
    end
end
```

## Libraries and Features

- 如非必要, 不要使用`debug library`, 尤其是运行可信代码(使用debug library有时是一种`hack`: [String Interpolation](http://lua-users.org/wiki/StringInterpolation))
- 避免使用过时(deprecated)的特性. 参考 [Lua 5.1 Reference Manual - 7 - Incompatibilities with the Previous Version](http://www.lua.org/manual/5.1/manual.html#7)

## Scope

无论何时, 尽可能的使用`local`

```
local x = 0
local function count()
    x = x + 1
    print(x)
end
```

全局(global) 有更大的范围和生命周期, 并且会增加[coupling](https://en.wikipedia.org/wiki/Coupling_(computer_programming))和复杂度. **不要污染环境**. 在Lua中, 由于全局(global)需要运行时查找table, 所以访问`local`比`global`更快, `local`保存在寄存器中. 参考[Scope Tutorial](http://lua-users.org/wiki/ScopeTutorial)

检测误用global的有效方法在[DetectingUndefinedVariables](http://lua-users.org/wiki/DetectingUndefinedVariables)给出. 在Lua中, `globals`有时可能导致错误拼写和潜在的错误.

有时通过`do-blocks`进一步限制`local变量`的范围非常有用

```
local v
do
    local x = u2 * v3 - u3 * v2
    local y = u3 * v1 - u1 * v3
    local z = u1 * v2 - u2 * v1
    v = {x, y, z}
end

local count
do
    local x = 0
    count = function() x = x + 1; return x end
end
```

## Mudules

在Lua 5.1版本中, 模块系统常被推荐, 但也有一些需要批评的地方. 更详细的描述可以看[Lua Module Function Critiqued](http://lua-users.org/wiki/LuaModuleFunctionCritiqued). 你可能写出一下代码:

```
-- hello/mytest.lua
module(..., package.seeall)
local function test() print(123) end
function test1() test() end
function test2() test1(); test1() end
```

并且像下方一样使用

```
require "hello.mytest"
hello.mytest.test2()
```

这里需要批判的地方是: 在所有模块中创建了一个全局变量`hello`(包含副作用), 通过hello table全局环境曝光. 如: `hello.mytest.print = _G.print`.

这些问题可以通过不适用`module`函数, 而是通过简单的定义模块来解决: 

```
-- hello/mytest.lua
local M = {}

local function test() print(123) end
function M.test1() test() end
function M.test2() M.test1(); M.test1() end

return M
```

然后通过如下方式import

```
local MT = require "hello.mytest"
MT.test2()
```

一个模块包含带构造函数的类可以通过多种方法封装. 此处提供一个好方法:

```
-- file: finance/BankAccount.lua

local M = {}; M.__index = M

local function construct()
    local self = setmetatable({balance = 0}, M)
    return self
end
setmetatable(M, {__call = construct})

function M:add(value) self.balance = self.balance + 1 end

return M
```

一个模块通过以上方式定义一般只有`一个类`, 这个类就是模块本身.

使用方式如下:

```
local BankAccount = require "finance.BankAccount"
local account = BankAccount()
```

或者是

```
local new = require
local account = new "finance.BankAccount" ()
```

## Commenting

单行注释在`--`后加`一个空格`

```
return nil  -- not found    (suggested)
return nil  --not found     (discouraged)
```

文档注释有多种写法. 可以使用下方的装饰器模式

```
local docstrings = setmetatable({}, {__mode = "kv"})

function document(str)
    return function(obj) docstrings[obj] = str; return obj end
end

function help(obj)
    print(docstrings[obj])
end

document[[Print the documentation for a given object]](help)
document[[Add a string as documentation for an object]](document)

f = document[[Print a hello message]](
    function()
        print("hello")
    end
)
f()
help(f)
```

**End终止符(End Terminator)**

在end后增加一些语句结束的注释发会帮助增加可读性

```
for i, v in ipairs(t) do
    if type(v) == "string" then
        ...lots of code here...
    end -- if string
end -- for each t
```

## Lua Idioms

- 测试一个变量不等于nil, 下方的写法更简洁. 

> Lua中nil和false做为`false`, 其他所有值为`true`

```
local line = io.read()
if line then  -- instead of line ~= nil
...
end
...
if not line then  -- instead of line == nil
...
end
```

但是当值即可能等于`nil`也可能等于`false`时, 条件需要明确.

- `and`和`or`的短路求值特性也可以简化代码

```
local function test(x)
    x = x or "idunno"
    -- rather than if x == false or x == nil then x = "idunno" end
    print(x == "yes" and "YES!" or x)
    -- rather than if x == "yes" then print("YES!") else print(x) end
end
```

- 克隆(clone)一个table(对表的大小有系统依赖限制)

```
u = {unpack(t)}
```

- 判断一个table是否为空

```
if next(t) == nil then ...
```

- 向数组中增加一个值

```
-- #t表示数组当前长度
t[#t + 1] = 1  -- 代替 table.insert(t, 1)
```

## Design Patterns

Lua是一个轻量级语言, 通过少量的模块支持可以提供大量的强大的方式. 这种自由的设计模式需要自我组织. 

> Lua is small language with a small number of simple building blocks that can be combined in a vast number of powerful ways. With this freedom comes the need for self-discipline in the form of design patterns. Often there is an idiomatic way or design pattern to achieving a certain effect in Lua that can be reused frequently (e.g. ObjectOrientationTutorial or ReadOnlyTables). See the LuaTutorial and SampleCode for common solutions to such problems.(这部分翻译不好, 放上原文)


## Coding Standards

在各种Lua项目中的编码标准:

- [Sputnik coding standard](http://sputnik.freewisdom.org/en/Coding_Standard)




## 参考链接

- [Lua编码规范](http://lua-users.org/wiki/LuaStyleGuide)
- [高性能 Lua 技巧（译）](https://segmentfault.com/a/1190000004372649)
- [Lua Performance Tips](http://www.lua.org/gems/sample.pdf)

