---
title: 开发机安装配置golang和使用CGI
date: 2016-12-13 14:46:08
tags: Golang
desc: CGI
---



本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

## 安装golang


- [Golang download](https://golang.org/dl/)下载合适版本的`golang`二进制发布包.

<!--more-->

1. Xshell5 登录跳板机->开发机
2. 执行`rz -bey`选中本地电脑中下载的golang压缩包
3. 执行以下命令解压并安装二进制文件到`/usr/local`中

```go
$ tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
```

4. 配置golang对应的环境变量


```go
# 使用vim修改对应的配置文件
$ vim /etc/profile
# 配置环境变量
export PATH=$PATH:/usr/local/go/bin
```

## 什么是cgi?

- Web服务器负责管理连接， 数据传输，网络交互
- CGI脚本负责管理具体的业务逻辑， **CGI是一种网页和程序通讯的协议**


> `CGI`程序负责生成动态内容， 然后返回给服务器， 由服务器转交给客户端。CGI是一个独立的程序可以独立运行



## 使用golang和cgi包

> golang开发时, 需要对当前工作目录配置 `GOPATH`.

```go
# 到工作目录下执行以下目录, 将当前目录设置为GOPATH
$ export "GOPATH=$PWD"
```


简单的机遇HTTP Server的cgi处理程序

```go
package main

import (
	"net/http"
	"net/http/cgi"
	"log"
	"fmt"
)

func CgiTestHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Println("get request...", r.URL.Path)
	handler := new(cgi.Handler)
	handler.Path = "/usr/local/go/bin/go"
	// 运行脚本位置
	script := "/root/AsyncLiu/Golang/CgiTest/script/" + r.URL.Path
	log.Println(handler.Path)

	handler.Dir = "/root/AsyncLiu/Golang/CgiTest/script/"
	args := []string{"run", script}
	handler.Args = append(handler.Args, args...)
	handler.Env = append(handler.Env, "GOPATH=/root/AsyncLiu/Golang/CgiTest")
	log.Println(handler.Args)

	handler.ServeHTTP(w, r)
}

func main() {
	http.HandleFunc("/", CgiTestHandler)
	fmt.Println("start web server....")
	log.Fatal(http.ListenAndServe(":8888", nil))
}

```

对应的被执行的脚本

```go
package main

import "fmt"

func init() {
	fmt.Println("Content-Type: test/plain;charset=utf-8\n\n")
}

func main() {
	fmt.Println("This is executing go script")
}

```


