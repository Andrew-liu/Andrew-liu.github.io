title: 快学Scala
date: 2016-01-16 00:09:04
tags: Scala
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


## Scala简介

`Scala` 是一门多范式的编程语言, 由`Martin Odersky` 于2001年基于Funnel的工作开始设计Scala并于2004年正式发布

- Scala是一种纯面向对象的语言，每个值都是对象
- Scala是一门多范式编程语言, 支持命令交互式, 函数式, 面向对象
- 编译型高性能语言(`静态`)
- **与Java无缝兼容, 可以使用任何Java库**

<!--more-->

## 代码风格

1. 函数和变量以`小驼峰命名`
2. 类和特质以`大驼峰命名`
3. 常量使用全大写命名
4. 一般使用两格缩进
5. Scala大部分情况可以忽略语句末尾的分号

## Scala变量

> Scala中尽量避免使用变量, 函数式编程的一个重要特性是`不可变性`(不可变变量没有副作用)

- Scala是静态类型语言, 但是不需要显式的指明变量类型, Scala采用类型推断(`Type Inference`)

```
//定义一个变量
var x = 0

// 定义
val y = 1
```


## Scala基本类型和操作

> String和值类型Byte, Short, Int, Long, Float, Double, Char, Boolean

- Scala的操作符不是特殊的语言语法, 任何方法都可以是操作符
- 操作符分为前缀, 中缀, 后缀
- Scala中所有操作符都是方法调用


```
# 前缀
scala> (2.0).unary_-
res1: Double = -2.0

# 中缀
scala> x indexOf 'o'
res0: Int = 4

# 后缀
scala> val x = "Hello, World"
x: String = Hello, World
scala> x.toLowerCase
res0: String = hello, world
```

- 中缀操作符的两个操作数, 一个在左一个在右
- 前缀操作符方法名在操作符上加了`unary_`前缀(`+, -, !, ~`)
- 后缀操作符是不用点或括号调用的不带任何参数的方法


算术操作符: `+, -, *, /, %`
关系, 逻辑和位操作: `>, <, >=, <=, ==, !=, &&, ||, &, |, ^, ~(反码)`
位移操作: `<<, >>, >>>(无符号右移)`



## Scala函数

> 函数式语言的一个主要特征是, 函数是第一类结构

**函数定义如下图:**

![函数定义](http://ww4.sinaimg.cn/large/ab508d3djw1eznwrn1zjcj20iv0d6taj.jpg)

- `Unit` 的结果类型指的是函数没有返回有用的值



## 函数式对象


> object和class的区别在于: object关键字创建一个单例对象

- 主构造器是类的`唯一入口`, 只有主构造器可以调用超类构造器
- `override`关键字用于在重载父类的非抽象成员和成员函数
- 同一个类内函数名相同而参数类型和个数不同的函数重载不需要`override`


```
class Rational (n: Int, d: Int) {

  //precondition
  require(d != 0)

  // 私有成员
  private val g = gcd(n.abs, d.abs)
  var numer: Int = n / g
  var denom: Int = d / g

  // auxiliary constructor, 相当于python中__init__构造函数
  def this(n: Int) = this(n, 1)
  // 函数重载
  override def toString = n + "/" + d;

  def add(other: Rational): Rational =
    new Rational(numer * other.denom + other.numer * denom, denom * other.denom)

  def -(other: Rational): Rational =
    new Rational(numer * other.denom - other.numer * denom, denom * other.denom)

  // 函数重载
  def -(i: Int): Rational =
    new Rational(numer - i * denom, denom)

  def *(other: Rational): Rational =
    new Rational(numer * other.numer, denom * other.denom)

  def lessThan(other: Rational): Boolean =
    this.numer * other.denom < other.numer * this.denom

  def max(other: Rational): Rational =
    if (lessThan(other)) other else this

  private def gcd(a: Int, b: Int): Int =
    if (b == 0) a else gcd(b, a % b)
}

var x = new Rational(1, 3);
var y = new Rational(5, 7);

println(x add y)
println(x * y)
println(x - 1)

// 隐式转换, 放在解释器方位内
implicit def intToRational(x: Int) = new Rational(x)
println(1 - x)
```


## 继承和多态

- 继承
- 多态和动态绑定特性

> 动态绑定的特性即父类指针可以指向子类对象, 通过父类指针调用成员方法时, 会查找实际所指向的对象, 然后调用对象的内的对应方法

![](http://ww4.sinaimg.cn/large/ab508d3djw1ezyaprbvcmj20qo0ecmyq.jpg)

```
val el: Element = new ArrayElement(Array("hello"))
val e2: ArrayElement = new LineElement("hello")
```


## 内建控制结构

> 表示式会产生一个值

- Scala中if是能返回值的表达式, Scala中没有三元操作符, 但通过`if (condition) var1 else var2` 可以实现三元操作符的功能
- while和do-while被称为`循环`, 不产生有意义的结果
- Scala中for语句非常强大, `for {子句} yield {循环体}`
- match表达式可以产生值, match远强大与其他语言中的`switch`, 而且不需要显示的声明break
- 变量范围: 大括号引入了一个新的范围, 内部变量会遮盖同名的外部变量
- 占位符语法
- 函数文本(匿名函数, 类似于python中的lamda)

![](http://ww3.sinaimg.cn/large/ab508d3djw1eznwvjaswlj20dv05bgmc.jpg)

```
var filename = if (!args.isEmpty) args(0) else "default.txt"


var filesList = (new File(".")).listFiles
// i <- 1 to 4(包含4), i <- 1 until 4 (不包含4)
for (file <- filesList
     if file.isFile;
     if file.getName.endsWith(".scala"))  // 过滤器使用分号隔开
    println(file)
    
var firstArg = if(args.length > 0) args(0) else ""
// 任何种类的常量和其他都可以作为case
firstArg match {
    case "text" => println("text")
    case _ => println("default")
}

// 匿名函数的写法(lambda)
scala> var someNumbers = List(1, 2, 3, 4)
scala> someNumbers.filter(x => x % 2 == 0)

// 占位符语法
scala> someNumbers.filter(_ % 2 == 0)

// 偏函数 partial funciton
scala> def sum(a: Int, b: Int, c: Int) = a + b + c
scala> val a = sum(1, _: Int, 3)
scala> a(2)

// 变长参数(Array[String])
scala> def echo(args: String*) = for(arg <- args) println(arg)
scala> echo("one")
scala> echo("one", "two")


class DefaultConstructor ( name:String , age:Int){
  def this(name:String){
    /*自定义构造器，必需首先调用默认构造器*/
    this(name , 24) ; 
  }
  def show(){
    println( name + "-->" + age ) ;
  }
}
```

## 柯里化(carry)

```
// 普通函数
def sum(x: Int, y: Int) = x + y

// 柯里化函数
def sum(x: Int)(y: Int) = x + y

// 实际执行
def sum(x: Int) = (y: Int) => x + y
```

## 特质(trait)

> 特质就像带有具体方法的java接口

> 特质和抽象类的区别: 抽象类主要用于有明确的父子继承关系的类树, 而特质可以用于任何类

特质定义使用`trait`关键字

```
trait Person() {
    def detail() {
        println("I'm angry!")
    }
}

// 使用extends或with混入特质, 从特质继承的方法可以像从超类继承的方法使用
class Student extends Person with Boy {
}
```

## 数据结构

- Python中常用`list, tuple, set, dict`
- Scala对应的数据结构为`List, Tuple[X], Set, Map(HashMap)`
- Scala中默认为不可变对象, 操作会生成一个新的对象


## 包

- Scala采用`java平台的包机制`
- 使用import来进行引用

```
package com.zhihu.antispam
```

## 参考链接 

- [Difference between object and class in Scala](http://stackoverflow.com/questions/1755345/difference-between-object-and-class-in-scala)
- [scala 入门](https://www.zybuluo.com/Great-Chinese/note/254972)
- [Scala 魔法函数](http://colobu.com/2016/01/04/Scala-magic-functions/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
- [Dependency Injection in Scala](http://blog.yunglinho.com/blog/2012/04/22/dependency-injection-in-scala/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
- [Databricks Scala 编程风格指南](http://www.hawstein.com/posts/databricks-scala-guide.html)
- [What are all the uses of an underscore in Scala?](http://stackoverflow.com/questions/8000903/what-are-all-the-uses-of-an-underscore-in-scala)

