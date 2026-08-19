title: Go goroutine同步
date: 2016-04-08 23:45:29
tags: Go
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


出现问题场景: 一个函数run()中包含多个goroutine函数并发, 这些goroutine函数会生成中间文件, 被run()函数运行结束后的check()函数检查. 当goroutine并发时, 并不会阻塞run()的上下文, 可能导致的情况为run()函数执行完毕(`但其中的goroutine并发函数没有执行完毕`), 导致check()函数执行失败.

**所以我们需要一种操作, 直到当前所有goroutine没有执行完毕, 才进行下一步操作**

> 所以需要`goroutine同步`, go提供了`sync包`和`channel机制`来解决goroutine之间的同步问题

<!--more-->

## sync.WaitGroup



```
A WaitGroup waits for a collection of goroutines to finish. The main goroutine calls Add to set the number of goroutines to wait for. Then each of the goroutines runs and calls Done when finished. At the same time, Wait can be used to block until all goroutines have finished.  -- 出自官方文档
```

大概意思是: `WaitGroup`等待一组goroutinue执行完毕. 主goroutinue调用`Add`设置等待的goroutinue数量. 每个goroutinue应该在执行结束时调用`Done`. `Wait`会阻塞知道所有goroutinue执行完毕.

> `WaitGroup`的用于某个地方需要创建多个goroutine，并且一定要等它们都执行完毕后再继续执行接下来的操作.

可以把`WaitGroup`看作一个类似任务队列的结构. Add想队列增加任务, Done完成任务, Wait在队列不空的时候阻塞在哪里.

```
// 官方文档中的example
package main

import (
    "fmt"
    "sync"
    "net/http"
)

func main() {
    var wg sync.WaitGroup  // 声明一个WaitGroup变量
    var urls = []string{
            "http://www.baidu.org/",
            "http://www.alibaba.com/",
            "http://www.qq.com/",
    }
    for _, url := range urls {
            wg.Add(1)  // WaitGroup的计数加1
            // Launch a goroutine to fetch the URL.
            go func(url string) {
                    defer wg.Done() //  goroutinue完成后, WaitGroup的计数-1
                    // Fetch the URL.
                    http.Get(url)
            fmt.Println(url);
            }(url)
    }
    // Wait for all HTTP fetches to complete.
    wg.Wait()  // 等待所有goroutinue完成
}
```


## channel

> channel同样可以用来同步goroutinue

**channel四种操作**

```
// make创建chennel, 第一个参数为channel的类型, 第二个参数为channel缓冲区的大小, 为0或者不传入该参数则表示没有缓冲区
exampleChannel := make(chan int, 100) 

// 放入数据到channel (channel <- data) 
exampleChannel <- 1

// 取出数据 (<-channel)
number := <-exampleChannel

// 关闭channel (通过close()函数)
close(exampleChannel)
```

channel是一种`阻塞管道`, 是自动阻塞的. 如果`channel`满了, 对channel放入数据的操作就会阻塞, 直到有某个routine从channel中取出数据, 这个放入数据的操作才会执行. 相反同理, 如果管道是空的, 一个从channel取出数据的操作就会阻塞，直到某个routine向这个channel中放入数据, 这个取出数据的操作才会执行(原理非常类似阻塞型socket的读写缓冲区)


```
// 上面的样例代码使用chan同步的方式来重写
package main

import (
    "fmt"
    "net/http"
)

func main() {
    var urls = []string{
            "http://www.baidu.org/",
            "http://www.alibaba.com/",
            "http://www.qq.com/",
    }
    doneChannel := make(chan int, len(urls))  // 创建channel
    for _, url := range urls {
            // Increment the WaitGroup counter.
            // Launch a goroutine to fetch the URL.
            go func(url string) {
                    // Decrement the counter when the goroutine completes.
                    // Fetch the URL.
                    http.Get(url)
                    doneChannel <- 1  // 向channel里放入数据表示完成操作
            fmt.Println(url);
            }(url)
    }
    // Wait for all HTTP fetches to complete.
    for i := 0; i < len(urls); i++{
        <-doneChannel  // 当可以取出len(urls)个数据时, 表示所有goroutinue都完成
    }
    fmt.Printf("Finish..\n")
}
```

**当任务的数量不固定**

```
package main

import (
    "fmt"
)


func test_chan(groutineChan chan int, feedbackChan chan string) {
    defer func() {
        <-groutineChan
        feedbackChan <- "finish"
    }()

    // do some process
    // ...
}

func main() {
    var (
        goroutineChan chan int    = make(chan int, 20)
        feedbackChan chan string = make(chan string, 10000)
        counter int
        finish int
    )
    for i := 0; i < 1000; i++ {
        goroutineChan <- 1
        counter++
        go test_chan(goroutineChan, feedbackChan)
    }

    for {
        msg := <-feedbackChan  // 从channel取出字符串
        if msg == "finish" {
            finish++  // 没完成一个完成计数器加一
        }
        if finish == counter { //当完全计数器等于计数器表示所有的goroutine完成
            break
        }
    }
    fmt.Printf("Finish..\n")
}

```

## 参考链接

- [go语言WaitGroup用法](http://www.baiyuxiong.com/?p=913)
- [golang学习笔记之—WaitGroup](http://studygolang.com/articles/4992)
