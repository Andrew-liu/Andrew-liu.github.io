title: shadowsocks源码剖析
date: 2016-07-24 22:09:02
tags: Go
---

> VPS上嗨皮的跑了那么久shadowsock的server了, 但一直比较想搞明白具体是怎么实现的. 前段时间比较忙, 最近总算有点私人时间了, 速度的强读一发shadowsocks源码

## 什么是shadowsocks?

- 不想再强行安利一发了, 毕竟开发者都被请去喝茶了
- 只需要知道和`G * F * W`有关就好了
- 如果上面你都不知道, 说明这篇文章与你无缘, 叉掉吧

所读源码基于[shadowsocks-go 1.1.5](https://github.com/shadowsocks/shadowsocks-go), 为什么选择golang实现版本? 我难道会告诉你, Golang的吉祥物太萌了, 我已经决定好好安利golang了. 正经点的回复, golang的语法灵活不失简洁, 棒棒哒.

<!--more-->

## 终入正题

先来张图, 假装自己很专业...

![](http://ww2.sinaimg.cn/large/ab508d3djw1f64vgco021j215m082jtz.jpg)


**流程概述:**

1. shadowsocks包含local和server两个程序, local运行在当前登录想要翻墙的机器上, server运行在墙外的服务器上(比如我的server运行在[vultr家的VPS](http://www.vultr.com/?ref=6844015)上)
2. local监控本地1080端口, 提供socks v5协议接口, 浏览器请求和local的1080端口建立TCP连接, 首先进行local端对本机进程进行心跳检测, 然后与server端建立请求发出实际请求包(`目标的地址和端口`)
3. 连接传输的数据通过加密,一般使用`aes-256-cfb`
4. server端解密收到的数据, 然后与实际请求的目标ip和端口建立TCP连接, 将获取的数据写回到local端, local端最终写回到浏览器进程完成一次翻越长城.

## show me your code

> 口说无凭, 也让人觉得云里雾里, 所以来是直接上代码吧

**PS: 代码均进行了精简, 只保持核心逻辑**

- 首先来看shadowsocks-local文件

```
// local的入口
func main() {
    /*
     * 1. 设置日志配置和命令行参数解析
     * 2. 从文件中读取相关配置, 如server ip, port, 用户设置的密码, 本地监控端口等
     */
    run(cmdLocal + ":" + strconv.Itoa(config.LocalPort))
}

// 实际监控函数
func run(listenAddr string) {
    /*
     * 1. 设置要监控的端口
     * 2. 在一个死循环中, 不断接收请求, 然后在goroutine中调用handleConnection(conn)处理请求
     */
    ln, err := net.Listen("tcp", listenAddr)
    for {  // 运行与一个死循环
        conn, err := ln.Accept()
        go handleConnection(conn)
    }
}

// 运行在goroutine中请求处理函数
func handleConnection(conn net.Conn) {
    /*
     * 1. 与请求进程进行心跳检测
     * 2. 解析请求获取原始数据, 用于获取实际请求ip和port
     * 3. 与墙外的server请求建立TCP连接, 并发送数据
     * 4. 等待墙外server写入数据, 然后将数据写回给请求进程
     */
    handShake(conn)
    rawaddr, addr, err := getRequest(conn)

    _, err = conn.Write([]byte{0x05, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x08, 0x43})
    remote, err := createServerConn(rawaddr, addr)

    go ss.PipeThenClose(conn, remote)
    ss.PipeThenClose(remote, conn)
}
```

最后我们来看一下 `PipeThenClose`这个函数的实现

```
// PipeThenClose copies data from src to dst, closes dst when done.
func PipeThenClose(src, dst net.Conn) {
    // closes dst when done.
    defer dst.Close()
    /*
     * 1. 从leaky buffer中获取空闲buffer
     * 2. 从source net.Conn中读取数据, 然后转发给dst net.Conn
     */
    buf := leakyBuf.Get()
    defer leakyBuf.Put(buf)
    for {
        SetReadTimeout(src)
        n, err := src.Read(buf)
        // read may return EOF with n > 0
        // should always process n > 0 bytes before handling error
        if n > 0 {
            // Note: avoid overwrite err returned by Read.
            if _, err := dst.Write(buf[0:n]); err != nil {
                Debug.Println("write:", err)
                break
            }
        }
        if err != nil {
            break
        }
    }
}
```


- 再来看shadowsocks-server代码, 你会发现两者惊人的相似

```
// 主要来handleConnection
// goroutine, 内部需要自行保证线程安全
func handleConnection(conn *ss.Conn, auth bool) {
    /* 
     * 1. 解析请求的ip:port
     * 2. 建立TCP连接
     * 3. 向目标站点发起请求, 响应数据写会到local
     */
    host, ota, err := getRequest(conn, auth)

    //对应ip:port建立TCP连接
    remote, err := net.Dial("tcp", host)

    if ota {
        go ss.PipeThenCloseOta(conn, remote)
    } else {
        go ss.PipeThenClose(conn, remote)
    }
    // 从remote读取数据
    ss.PipeThenClose(remote, conn)
    closed = true
    return
}
```

由local和server端总共四个`PipeThenClose`函数完成了由local到server, 再由server到local的数据传输. 但是要注意的是在`server端是没有与local进行心跳检测的阶段`


## socks v5

socks v5代码TCP协议的流程:

1. 用户进程与监听1080端口的进程建立TCP连接
2. 心跳检测, 用户进程向监听进程发送`VER, NMETHODS, METHODS`, 监听进程向请求进程响应`VER, METHOD`
3. 用户进程结束心跳检测, 向监听进程发起请求, 格式如下
4. 监听进程对用户进程响应, 表示成功, 则表示用户进程可以进行发送数据.

```
向监听进程的请求:
　  +----+-----+-------+------+----------+----------+
　　|VER | CMD |　RSV　| ATYP | DST.ADDR | DST.PORT |
　　+----+-----+-------+------+----------+----------+
　　| 1　| 　1 | X'00' | 　1　| Variable |　　 2　　|
　　+----+-----+-------+------+----------+----------+
监听进程的响应: 
　　+----+-----+-------+------+----------+----------+
　　|VER | REP |　RSV　| ATYP | BND.ADDR | BND.PORT |
　　+----+-----+-------+------+----------+----------+
　　| 1　|　1　| X'00' |　1 　| Variable | 　　2　　|
　　+----+-----+-------+------+----------+----------+

// 对应shadowsocks-local中的:
_, err = conn.Write([]byte{0x05, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x08, 0x43})
```



## 参考链接

- [shadowsocks-go 1.1.5](https://github.com/shadowsocks/shadowsocks-go)
- [socks v5 协议的 RFC](https://www.ietf.org/rfc/rfc1928.txt)
