title: Scala模式匹配和Try
date: 2016-02-07 19:52:22
tags: Scala
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


> 2015年最后一天来一发.

从面向对象语言转来学习函数式语言, 疑惑还是非常多的..

- 没有`return`关键字, 默认的最后执行结果返回
- 迷一般的模式匹配
- `map(映射)`语法(虽然之前在Python中用过类似的)
- 表达式和关键字区别是非常大的.
- 还有至今不懂的`monad`

<!--more-->

> 模式匹配是数据结构中字符串的一种基本运算, 给定一个子串, 要求在某个字符串中找出与该子串相同的所有子串, 而在Scala中, 模式匹配可以认为针对各种数据结构进行匹配


## 比switch更强大

最开始我以为模式匹配用来替代其他语言中的Switch语句的, 后来我发现这个认识太浅薄了, 模式匹配远比Switch强大太多

- 不需要明确的在每个逻辑最后追加`break`, 默认是逻辑匹配后自动break
- `case _`替代其他语言中的`default`语句. 如果没有不加这句, 比呢器没有匹配的模式, 会抛出`MatchError`异常
- match表达式可以匹配任意类型的变量
- match不是语句而是`表达式`, `表达式本身是有返回值的!`

```
def originMatch(name: String) = name match {
    case "Andrew" => println("My name is $name")
    case "Andrewliu" => println("He name is $name")
    case "Liu" => println("helloworld")
    case _ => println("Everything")
  }
```


## 捕获异常与Try

scala中的`try-catch`语句写起来和其他语言是差不多的, 其中异常的处理可以通过case进行具体的异常匹配,然后执行对应的逻辑

```
def tryCatch() = {
    try {
      1 / 0
    } catch {
      case e: ArithmeticException => println("分母为0")
    } finally {
      println("finally")
    }
  }
```

`try-catch`是面向对象语言的常用形式, 那么更加函数式的异常捕获呢? `使用Try`

Try有两个子类型:

1. `Success[A]`: 代表成功的计算
2. 封装了`Throwable`的`Failure[A]`, 代表出错的计算

如果当前处理逻辑可能导致异常, 可以简单使用`Try[A]`作为函数的返回类型

```
def tryMethod(name: String): Try[String] = {
    Try {
      println("My name is $name")
      throw new IOException("IOException")
    } match {
      case Success(result) => Try("hello world")
      case Failure(e) => println("hello world, This is a IOException"); Try("This is a Exception")
    }
  }
```

此处Success是当Try中逻辑正行执行时执行, 返回为Failure[String], Failure是在Try逻辑中出现异常时, 返回值编程了Failure[String]


## 类型匹配

在Python中使用isinstance来判断变量的类型, 而且必须要要记住对应的`type`, 每次都要查手册才行.

模式匹配来进行变量类型的匹配是极为感人的, 并且可以对匹配的变量进行重新命名, 简直感人.

类型匹配发生在`运行期`

```
def patternMatch(para: Any): Unit = {
    para match {
      case i: Int if i < 0 => i - 1
      case i: Int => i + 1
      case d: Double if d < 0.0 => d - 0.1
      case d: Double => d + 0.1
      case text: String => text + "s"
      case _ => println("Any")
    }
  }
```

> 注意case语句中是可以有`if语句对变量进行判断`, 如果在case后跟着一个变量名, 当条件匹配后, 会将值赋值给这个变量, `比如上面的case i: Int if i < 0匹配后, para会赋值给i`




## 样式类与模式匹配的结合

`模式匹配` 解绑一个给定的数据结构, 这都归功于`提取器`.

提取器的功能与`构造器`相反, 提取器从传递给它的对象中提取出构造对象的参数. 在`单例对象`使用unapply来提取固定数量的对象


```
// 使用unapply方法来提取参数, 当提取多个值时Option中应该包含多个类型
class MyExtractor(val name: String, val male: String)
object  MyExtractor {
  def unapply(extractor: MyExtractor): Option[String] = Some(extractor.name)
}

// 调用
def useUnapply(extractor: MyExtractor): String = {
    case MyExtractor(name, male) => "hello " + name
    case _ => "Not unapply!"
  }
  
// 样本类, 不需要new关键字来创建对象, 并且有构造函数, 并且被设计用于模式匹配
case class Calculator(brand: String, model: String)

def caseClass(calc: Calculator) = calc match {
    case Calculator("hp", "32G") => "good"
    case Calculator("apple", "25G") => "better"
    case Calculator("xiaomi", "1G") => "bad"
    case Calculator(_, _) => "hello world"
  }
```


样式类经过优化, 用于模式匹配, 声明样式类时:

- 构造器的每个参数默认设置为val 
- 在半生对象中`提供了apply和unapply`方法(定义apply方法后, 可以直接通过通过类名调用对象, 定义unapply用于模式匹配)



## 参考链接

- [Scala 初学者指南](https://windor.gitbooks.io/beginners-guide-to-scala/content/chp6-error-handling-with-try.html)
- [http://hongjiang.info/scala-pattern-matching-1/](http://hongjiang.info/scala-pattern-matching-8/)
