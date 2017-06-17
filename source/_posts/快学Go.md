title: 快学Go
date: 2016-03-27 22:13:09
tags: Go
---

## 简介

- `Go`是Google开发的一种静态强类型、编译型，并发型，并具有垃圾回收功能的编程语言
- 面向对象三大特性(封装/继承/多态), Go语言仅仅支持封装

```go
// 使用go run test.go, 运行下面源码
package main
import "fmt"

func main() {
    fmt.Println("Hello, World")
}
```

<!--more-->


## 类型

- 常用类型`bool byte(uint8) string int(类型多, 就不一一列出了) float(float32, float64) complex(complex64, complex128)`

- var 语句定义了一个变量的列表

```go
// 变量定义语法: var var_name type
var andrew string
var liu string = "andrewliu"

// 段声明语法只能用在函数内
k := 3
// 表达式 T(v) 将值 v 转换为类型 T, Go不支持隐式转换! 必须显示转换类型
var f float32 = 3.2222
var z uint = uint(f)
// 定义一个常量
const World = "世界"
```

## 控制流

### if语句


- if语言的condition不需要小括号
- if语句的左大括号不能另起一行
- 可以在if语句中定义局部变量, 生命周期在if-else内

```go
// if语句后没有小括号, 但是逻辑题要使用{}
if v := math.Pow(x, n); v < lim {
        return v  // v的生命周期在if-else语句内部
    } else {
        fmt.Printf("%g >= %g\n", v, lim)
    }
```

### switch语句

- switch语句不需要显式的使用`break`, 满足条件后会自动终止

```go
func main() {
    t := time.Now()
    switch {  // switch后可以没有语句, 类似于if-else if- else
    case t.Hour() < 12:
        fmt.Println("Good morning!")  //满足条件后会自动退出
    case t.Hour() < 17:
        fmt.Println("Good afternoon.")
    default:
        fmt.Println("Good evening.")
    }
}
```

### 循环



```go
// 类似于c语言, 只是没有小括号, 循环体使用{}
import "fmt"

func main() {
    sum := 0
    for i := 0; i < 10; i++ {
        sum += i
    }
    fmt.Println(sum)
}

// c语言中的while语句, 在Go依然为for关键字
for sum < 1000 {
        sum += sum
    }
    
//  while(true)语句
for {
    fmt.Println("andrewliu")
}
```

值得一提的是`range`, 类似迭代器操作, 返回`(索引, 值)`或 `(键, 值)`

```go
// range操作类似于python中的enumerate
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}
func main() {
    for i, v := range pow {
        fmt.Printf("2**%d = %d\n", i, v)
    }
}
```

## 指针

> Go竟然有指针, 我简直惊呆了

- 不支持指针运算, 不支持 `->` 运算符, 直接`.`访问成员


```go
    i, j := 42, 2701
    p := &i         // p指针指向变量i
    fmt.Println(*p) // p解引用操作, 读取内存地址上的值
    *p = 21         // 通过指针来设置i的值
    fmt.Println(i)
```

## 数据结构


### 结构体


- 使用`type`定义新的结构体
- `值类型`, 赋值和传参会复制全部内容
- Go语言中没有类和继承的概念, 类似于类的机制可以通过struct来实现
- struct支持嵌套

```go
type Vertex struct {
    X float64
    Y float64
}
```

**方法(可以理解为类的成员函数)**

1. Go没有类, 但可以通过结构体定义方法, 方法接收者`receiver`是出现在`func`关键字和方法名之间的参数
2. 方法不支持重载
3. receiver可以为`结构体或者结构体指针`

```go
func (v *Vertex) Abs() float64 {
    return math.Sqrt(v.X * v.X + v.Y * v.Y)
}
func main() {
    fmt.Println(Vertex{1, 2})
}
```

### 数组

- 数组是值类型, 赋值和传参会复制整个数组
- 支持多维数组

```go
// 定义一个长度为10的定长数组
var a [10]int
// 通过初始化的值来确定数组长度
var b [...]int{1, 2, 3, 4}
```


- `slice`是引用类型
- `slice`并不是数组或数组指针, slice通过`内部指针`和相关属性(如长度, 容量)引用数组片段, 从来实现变长数组
- 读写操作实际目标是`底层数组`
- 当slice容量满后, append后会重新分配底层数组, 并复制数据(很想C++中`vector`), 通常以2被容量重新分配底层数组


```go
// 直接创建一个slice, 支持切片操作
s := []int{2, 3, 5, 7, 11, 13}
// 使用make函数构造一个slice
a := make([]int, 5)

// 第一个参数为slice, 其余的元素被添加到slice尾部, 最终返回新slice对象
func append(s []T, vs ...T) []T
```



### map


- 引用类型, 底层数据结构为哈希表(O(1)查询时间复杂度)
- 键必须是支持相等运算符`(==、!=)`类型, 如 number、string、 pointer、array、struct,以及对应的 interface
- 不保证key的顺序性
- 从map中得到的value是value的复制(可以通过将value值设为指针来直接修改数据)

```go
var m map[string]Vertex
// map需要通过make创建, []中为key的类型, 后面为value的类型
m = make(map[string]Vertex)

var m = map[string]Vertex{
    "Bell Labs": Vertex{
        40.68433, -74.39967,
    },
    "Google": Vertex{
        37.42202, -122.08408,
    },
}

// 判断是否key存在在map中
if v, ok := m["Apple"]; !ok {
    // key不存在则进入此逻辑
}

// 遍历一个map
for k, v := range m {
    // some process for key-value
}
```

## 函数

-  支持不定长变参, `不支持默认参数`
-  可以有多个返回值
-  函数可以作为参数传递

```
//  函数定义
func add(x int, y int) int {
    return x + y  // 可以有多个返回值
}
```


```
package main
//  通过()一次导入多个包
import (
    "fmt"
    "time"
)
func compute(fn func(float64, float64) float64) float64 {
    return fn(3, 4)
}
func main() {
    fmt.Println("The time is", time.Now())
    fmt.Println(" My name is Liubin")
}
```
 
**函数作为参数**

```go
type FormatFunc func(s string, x, y int) string // 定义函数类型。
func format(fn FormatFunc, s string, x, y int) string {
    return fn(s, x, y)
}
```

**可变参数**, 只能有一个, 必须位于参数表最后.

```go
// 可变参数本质上是一个slice,
func test(s string, n ...int) string {
    var x int
    for _, i := range n {
        x += i
    }
    return fmt.Sprintf(s, x)
}
```

**匿名函数**

```go
// 函数内设置一个匿名函数变量
fn := func() { println("Hello, World!") }
fn()
```

## defer

关键字`defer` 用于注册延迟调用, **defer后跟的语句知道return前才被调用**

```go
package main

import (
    "os"
    "log"
)

func test() error {
    file, err := os.Create("test.txt")
    if err != nil {
        log.Fatal("create file err: ", err)
    }
    defer file.Close()
    file.WriteString("Hello, World")
    return  nil
}

func main() {
    test()
}
```

## 接口

> 接口是一个或多个`方法签名`的集合, 任何类型的方法集中只要拥有对应的全部方法, 就表示`实现`了该接口, `无需显示添加接口声明` --摘自<Go学习笔记>

```go
// 接口没有数据字段, 没有方法实现
type Stringer interface {
    String() string
}
```


## 错误处理

`error`是一个`build-in`的接口

```go
type error interface {
    Error() string
}
```

如何使用错误处理呢?

```go
func learnErrorHandling() {
    // ", ok"用来判断有没有正常工作 
    m := map[int]string{3: "three", 4: "four"}
    if x, ok := m[1]; !ok { // ok 为false，因为m中没有1
        fmt.Println("no one there")
    } else {
        fmt.Print(x) // 如果x在map中的话，x就是那个值喽。
    }
    // 错误可不只是ok，它还可以给出关于问题的更多细节。
    if _, err := strconv.Atoi("non-int"); err != nil { // _ discards value
        // 输出"strconv.ParseInt: parsing "non-int": invalid syntax"
        fmt.Println(err)
    }
}
```

## 并发

Go 提供并发特性, 类似于协程的`goroutine机制`(语言层面特性)

> 官方博客中对goroutinue的解释: A goroutine has a simple model: it is a function executing concurrently with other goroutines in the same address space. `在同一块地址空间多个goroutinue并发执行函数.`


- 在函数调用语句前加一个`go关键字`, 就可以实现goroutine并发
- 多个goroutinue执行次序无法保证
- channel用于多个goroutinue通信, `内部实现了同步来确保并发安全`. **默认为同步模式, 需要发送和接收配对. 否则被阻塞**


```go
func test_goroutinue() {
    data := make(chan int)  // 数据交换队列
    exit := make(chan bool)  //退出通知
    go func() {
        for d := range data {  // 从队列迭代接收数据, 直到close
            fmt.Println(d)
        }

        fmt.Println("recv over.")
        exit <- true  // 发送退出通知
    }()

    data <- 1  // 发送数据
    data <- 2
    data <- 3
    close(data)

    fmt.Println("send over.")
    <- exit  // 等待退出通知
}
```


## 参考链接

- [Go学习笔记](https://github.com/qyuhen/book)
- [Learn Go in Y minutes](https://learnxinyminutes.com/docs/go/)
- [A tour of the Go](https://tour.golang.org/welcome/1)
- [Go 语言的错误处理机制是一个优秀的设计吗？](https://www.zhihu.com/question/27158146)
- [Errors are values](http://blog.golang.org/errors-are-values)
- [Effective Go - Concurrent](https://golang.org/doc/effective_go.html#concurrency)

