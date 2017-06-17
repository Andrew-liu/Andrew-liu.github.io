title: How to Write Go Code
date: 2016-05-08 21:42:52
tags: Go
---


> 官方How to Write Go Code的阉割版 :-D

## GOPATH

`GOPATH环境变量`指明当前工作区的位置, 这可能在你编写Go程序时`唯一一个`需要设置的环境变量

```
# 设置GOPATH
$ mkdir example
$ cd example
# 类Unix设置环境变量, 只对当前session有效, 如需长期生效需要设置如.zshrc的配置文件
$ export "GOPATH=$PWD"
# 验证是否设置成功
$ echo $GOPATH
```

<!--more-->

## Go命令

`go install`第一步先生成结果文件(可执行文件或者.a包), 第二步会把编译好的结果移到`$GOPATH/pkg`或者`$GOPATH/bin`

```
# 生成命令行可执行文件
go install andrewliu/hello
```

`go build`主要用于编译代码

```
# 编译andrewliu/stringutil下的源码文件
go build andrewliu/stringutil
```

go继承轻量级测试, 测试文件的文件名为`*_test.go`的形式, 其中函数命名为`func TestXXX(t *testing.T)`, 如果函数调用失败的函数, 如`t.Error或者t.Fail`, 则认为这个测试用例失败

```
# 运行ndrewliu/stringutil下的*_test.go文件
$ go test andrewliu/stringutil
```
go语言有一个获取远程包的工具就是`go get`(类似于pip install),目前`go get`支持多数开源社区(例如: github、googlecode、bitbucket、Launchpad)

```
# 从github中拉取远程package
$ go get github.com/golang/example/hello
```

`go get -u`参数可以自动更新包，而且当`go get`的时候会自动获取该包依赖的其他第三方包

## 命名

- package后的包名使用小写, 不需要使用大小写混用或者下划线
- 自定义Get和Set方法时(Go不提供自动支持), 如果owner是个结构体, Get方法可以命名为`Owner`, Set方法命名为`SetOwner`
- 接口命名应该以`er`为后缀
- 大写字母开头的变量是可导出的, 也就是其它包可以读取的, 是`公有变量`, 小写字母开头的就是不可导出的，是`私有变量`
- 大写字母开头的函数也是一样, 相当于class中的带public关键词的`公有函数`, 小写字母开头的就是有private关键词的私有函数。


## 资源分配

- `new`是一个用来分配内存的内建函数,但是不像在其它语言中, 它并不初始化内存, 只是将其置零. 也就是说, new(T)会为T类型的新项目, 分配被置零的存储, 并且`返回它的地址`, 一个类型为*T的值. 在Go的术语中, 其返回一个指向新分配的类型为T, 值为零的指针

> 表达式`new(Type)`和`&Type{}`是等价的

- 内建函数`make(T, args)`与new(T)的用途不一样. 它只用来创建`slice、map和channel`, 并且返回一个初始化的(而不是置零), 类型为`T`的值（而不是*T）, 导致这三个类型有所不同的原因是指向数据结构的引用在使用前必须被初始化. 对于`slice、map和channel`来说, make初始化了内部的数据结构, 填充适当的值.


## 方法

任何命名类型（指针和接口除外）都可以定义方法(方法不是函数), 此时这个类型被称为接收器(receiver).

- 当接收器为值类型时, 方法对值的改变外部无法感知.

```
# 使用type给类型增加别名, 增加代码可读性
type ByteSlice []byte

# 接口器为[]byte类型, 方法内部可以使用slice
func (slice ByteSlice) Append(data []byte) []byte {
    // 一些对slice的修改
}
```

- 当接收器为指针时, 方法内部可以对接收器作修改, 外部可以感知.

```
# 接收器为指针
func (slice *ByteSlice) Append(data []byte) []byte {
    // 一些对slice的修改
}
```


## 接口

> interface是一组method`签名`的组合

- 任意的类型都实现了`空interface(类型为interface{})`, `空interface`作为函数参数时, 表示可以传入任意类型的变量
- `一个类型不需要明确的声明它实现了某个接口`.一个类型要实现某个接口, 只需要实现该接口对应的方法就可以了. 在实际中, 多数接口的类型转换和检查都是在编译阶段静态完成的. 例如, 将一个`*os.File`类型传入一个接受`io.Reader`类型参数的函数时, 只有在`*os.File`实现了`io.Reader`接口时，才能编译通过

假设我们只是想知道某个类型是否实现了某个接口, 而实际上并不需要使用这个接口本身 —— 例如在一段错误检查代码中—— 那么可以使用空白标识符(`_`)来忽略类型断言的返回值：

```
# 判断某变量是否实现了json.Marshaler接口(包含marshaling方法)
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}

# 判断某类型是否实现了接口(会触发编译器编译器的类型检查)
var _ json.Marshaler = (*RawMessage)(nil)
```

- 反向获取interface变量里面实际保存了的是哪个类型的对象

```
import (
    "fmt"
    "strconv"
)

type Element interface{}
type List []Element

type Person struct {
    name string
    age int
}

//定义了String方法，实现了fmt.Stringer
func (p Person) String() string {
    return "(name: " + p.name + " - age: "+strconv.Itoa(p.age)+ " years)"
}

func main() {
    list := make(List, 3)
    list[0] = 1 // an int
    list[1] = "Hello" // a string
    list[2] = Person{"Dennis", 70}

    for index, element := range list {
        if value, ok := element.(int); ok {
            fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
        } else if value, ok := element.(string); ok {
            fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
        } else if value, ok := element.(Person); ok {
            fmt.Printf("list[%d] is a Person and its value is %s\n", index, value)
        } else {
            fmt.Printf("list[%d] is of a different type\n", index)
        }
    }
}
```


## 并发

> goroutine运行在相同的地址空间, 而Go以channel通信的方式来实现共享变量, 

- 无缓冲区channel(`make(chan int)`), 只能想channel中传入一个数据, 此时传入第二个数据会阻塞, 必须等第一个被取出, 第二个数据才能成功传入
- 有缓冲区channel(`make(chan int, 5)`). 前五个数据可以无阻塞的传入, 第六个数据必须等前五个数据中的某个数据被取走才能成功传入, 否则会阻塞在哪里.
- channel的阻塞机制可以考虑与阻塞socket对应的读写缓冲区对比.


```
// 使用goroutine和channel来实现并发
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```

`<-c`会一直阻塞, 直到有数据可以从channel中取出.

> 在目前的Go runtime 实现中, goroutine代码在默认情况下是不会被并行化的。对于用户态任务, 默认仅提供一个物理CPU进行处理. 任意数目的goroutine可以阻塞在系统调用上, 但默认情况下, 在任意时刻,只有一个goroutine可以被调度执行. 未来go语言的设计可能更加智能, 但是目前, 必须通过设置`GOMAXPROCS环境变量`或者`导入runtime包并调用runtime.GOMAXPROCS(NCPU)`,  来告诉Go的运行时系统最大并行执行的Goroutine数目. 可以通过runtime.NumCPU()获得当前运行系统的逻辑核数, 作为一个有用的参考.


- **当存在多个channel时, 使用select监听channel上的数据流动**

> select默认是阻塞的, 只有当监听的channel中有发送或接收可以进行时才会运行, 当多个channel都准备好的时候, select是随机的选择一个执行的

```
package main

import "fmt"

func fibonacci(c, quit chan int) {
    x, y := 1, 1
    for {
        select {
        case c <- x:  // 向channel发送数据
            x, y = y, x + y
        case <-quit:
            fmt.Println("quit")
            return
        }
    }
}

func main() {
    c := make(chan int)
    quit := make(chan int)
    go func() {
        for i := 0; i < 10; i++ {
            fmt.Println(<-c)  // 取出fibonacci 传入的数据
        }
        quit <- 0
    }()
    fibonacci(c, quit)
}

```


## 参考链接

- [How to Write Go Code](https://golang.org/doc/code.html)
- [Effective Go](https://golang.org/doc/effective_go.html)
- [2.7 并发](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.7.md)

