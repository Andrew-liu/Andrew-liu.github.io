title: Golang RPC
date: 2016-07-02 23:18:02
tags: Go
---


本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


> RPC(Remote Procedure Call)是`分布式系统`中的一个关键机制.


## Go HTTP RPC

<!--more-->

```go
// 基于官方的example
package main

import (
    "fmt"
    "net"
    "net/rpc"
    "log"
    "net/http"
)

type Args struct {  // 参数封装
    A, B int
}

type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
    *reply = args.A * args.B
    return nil
}

func makeService() {
    /*
     func Register(rcvr interface{}) error
     将Service(Arith)注册到默认的Server中
      */
    rpc.Register(new(Arith))
    /*
     func HandleHTTP()
     RPC消息有HTTP Handler来处理, Handler注册到默认的Server
      */
    rpc.HandleHTTP()
    /*
     func Listen(net, laddr string) (Listener, error)
     第一个参数为面向流的网络("tcp", "tcp4", "tcp6", "unix", "unixpacket").
     第二个参数为字符串形式的地址, 如"127.0.0.1:1234", 省略host表示可以监听所有host
      */
    l, e := net.Listen("tcp", ":1234")
    if e != nil {
        log.Fatal("listen error:", e)
    }
    /*
    func Serve(l net.Listener, handler Handler) error
     */
    go http.Serve(l, nil)
}

func makeClient() {
    /*
    func DialHTTP(network, address string) (*Client, error)
    参数类似于net.Listen
     */
    client, err := rpc.DialHTTP("tcp", "127.0.0.1" + ":1234")
    defer client.Close()  // 延迟关闭client
    if err != nil {
        log.Fatal("dialing:", err)
    }
    args := &Args{7, 8}
    var reply int
    /*
    func (client *Client) Call(serviceMethod string, args interface{}, reply interface{}) error
    client的方法, 第一个参数为调用服务的方法, 第二个参数方法需要的参数, 第三个参数为执行结果
     */
    err = client.Call("Arith.Multiply", args, reply)
    if err != nil {
        log.Fatal("Arith rpc call error:", err)
    }
    fmt.Println(reply)
}

func main() {
    // 只是为了一个启动使用了goroutine，正常情况应该分别启动一个客户端和一个服务器
    makeService()
    makeClient()
}
```


## Go JSON RPC

```go
package main

import (
    "fmt"
    "net"
    "net/rpc"
    "log"
    "net/http"
    "net/rpc/jsonrpc"
)

type Args struct {  // 参数封装
    A, B int
}

type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
    *reply = args.A * args.B
    return nil
}

// 使用json作为rpc
func makeJSONService() {
    rpc.Register(new(Arith))
    tcpAddr, err := net.ResolveTCPAddr("tcp", ":1234")
    if err != nil {
        panic(err)
    }
    /*
     ListenTCP 类似于 Listen. func ListenTCP(net string, laddr *TCPAddr) (*TCPListener, error)
     第一个参数为面向流的网络("tcp", "tcp4", or "tcp6").
     第二个参数为字符串形式的地址, 如"127.0.0.1:1234", 省略host表示可以监听所有host
    */
    listener, err := net.ListenTCP("tcp", tcpAddr)
    if err != nil {
        panic(err)
    }
    go func () {
        for {
            conn, err := listener.Accept()
            if err != nil {
                continue
            }
            // 注意此处的不同
            /*
            func ServeConn(conn io.ReadWriteCloser)
            此函数阻塞运行一个JSON-RPC
             */
            jsonrpc.ServeConn(conn)
        }
    }()
}

func makeJSONClient() {
    // JSON-rpc的客户端基本和TCP一样
    client, err := jsonrpc.Dial("tcp", "127.0.0.1:1234")
    if err != nil {
        log.Fatal("dialing:", err)
    }
    // Synchronous call
    args := &Args{17, 8}
    var reply int
    err = client.Call("Arith.Multiply", args, &reply)
    if err != nil {
        log.Fatal("Arith rpc call error:", err)
    }
    fmt.Println(reply)
}
```

## 6.824 labrpc结构

**数据结构**:

- `Network`用于模拟网络, 使整个RPC调用过程中在内部发生, 不需要真正的执行网路请求
- `Server`一个RPC Server, 内部包含多个注册的Service
- `ClientEnd`表示RPC客户端的数据结构
- `Service`用于客户端来注册struct, 然后封装入该数据结构
- `reqSrc` RPC调用的参数被解析然后重新封装到整个数据结构中
- `replyMsg`封装RPC调用的返回结果

**架构(可以选择性忽略Network部分)**


![](http://ww4.sinaimg.cn/large/ab508d3djw1f2rxpft969j20i30c176z.jpg)




## 参考链接

- [RPC是什么](http://blog.brucefeng.info/post/what-is-rpc)
- [6.824 2016 Lecture 2: Infrastructure: RPC and threads](https://pdos.csail.mit.edu/6.824/notes/l-rpc.txt)
