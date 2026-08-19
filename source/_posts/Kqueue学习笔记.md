title: Kqueue学习笔记
date: 2016-08-14 10:33:53
tags:
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


> 想在Mac上造点小轮子, 然而epoll是在Linux平台独有的, 所以想到了用kqueue来替代. 记录一下自己的学习过程.

## Introduction

- `kqueue`是在UNIX上高效的IO复用技术, 类比于linux平台中的`epoll`.
- IO复用原理大概为: 网卡设备对应一个中断号, 当网卡收到网络端的消息的时候会向CPU发起中断请求, 然后CPU处理该请求. 通过驱动程序 进而操作系统得到通知, 系统然后通知epoll/kqueue, epoll/kqueue通知用户代码. 

<!--more-->

## API

kqueue使用的头文件和api可以通过`man kqueue`来看到

```
#include <sys/types.h>
#include <sys/event.h>
#include <sys/time.h>
```

- `kqueue()`系统调用会创建一个新的内核消息队列并返回描述符.

```
# 创建失败返回-1, 否则返回描述符
int kqueue(void);
```



- kqueue的数据结构

```
struct kevent {
    uintptr_t       ident;          /* identifier for this event, 该事件关联的文件描述符 */
    int16_t         filter;         /* filter for event */
    uint16_t        flags;          /* general flags, 用于指定事件操作类型, 比如EV_ADD, EV_ENABLE, EV_DELETE等, 通过|可以同时设置多个事件 */
    uint32_t        fflags;         /* filter-specific flags */
    intptr_t        data;           /* filter-specific data */
    void            *udata;         /* opaque user data identifier */
};
```

- `EV_SET()`在官方文档描述是一个宏,  用于初始化kevent数据结构

```
EV_SET(&kev, ident, filter, flags, fflags, data, udata);
// 使用范例, 将监听stdin描述符的可读事件初始化到ev数据结构
kevent ev;
EV_SET(&ev, STDIN_FILENO, EVFILT_READ, EV_ADD, 0, 0, 0);
```

- `kevent`为核心函数, 初始时`kqueue`内核消息队列为空, 使用kevent进行事件填充, 在不设置超时参数时, 只有当收到某监听事件才会返回. 该函数返回接收到事件个数, 并将事件写入eventlist

```
# changelist为要注册的事件列表, eventlist用于返回已经就绪的事件列表
int kevent(int kq, const struct kevent *changelist, int nchanges, struct kevent *eventlist, int nevents, const struct timespec *timeout);
```


## Example


```
#include <stdio.h>          // fprintf
#include <sys/event.h>      // kqueue
#include <netdb.h>          // addrinfo
#include <arpa/inet.h>      // AF_INET
#include <sys/socket.h>     // socket
#include <assert.h>         // assert
#include <string.h>         // bzero
#include <stdbool.h>        // bool
#include <unistd.h>         // close

const size_t BUF_SIZE = 1024;
static bool s_stop = true;
// 信号处理函数
static void handle_signal(int sig) {
    s_stop = false;
}

int learn_kqueue(const char* ip, int32_t port) {
    std::cout << "ip: " << ip << " port: " << port << std::endl;
    signal(SIGTERM, handle_signal);
    int sock = socket(PF_INET, SOCK_STREAM, 0);
    assert(sock > 0);

    // 强制使用TIME_WAIT状态的socket地址
    int reuse = 1;
    setsockopt(sock, SOL_SOCKET, SO_REUSEADDR, &reuse, sizeof(reuse));

    struct sockaddr_in address;
    bzero(&address, sizeof(address));
    address.sin_family = AF_INET;
    inet_pton(AF_INET, ip, &address.sin_addr); //主机序转网络序ip
    address.sin_port = htons(port); //主机序转网络序

    int ret = bind(sock, (struct sockaddr*)&address, sizeof(address));
    assert(ret != -1);
    ret = listen(sock, BACK_LOG);
    assert(ret != -1);

    //创建一个消息队列并返回kqueue描述符
    int kq =  kqueue();
    assert(kq != -1);

    struct kevent change_list[10];  //想要监控的事件
    struct kevent event_list[10];  //用于kevent返回
    char buf[BUF_SIZE];
    // 监听sock的读事件
    EV_SET(&change_list[0], sock, EVFILT_READ, EV_ADD | EV_ENABLE, 0, 0, 0);
    // 监听stdin的读事件
    EV_SET(&change_list[1], fileno(stdin), EVFILT_READ, EV_ADD | EV_ENABLE, 0, 0, 0);
    int nevents;
    while (s_stop) {
        printf("new loop...\n");
        // 等待监听事件的发生
        nevents = kevent(kq, change_list, 2, event_list, 2, NULL);
        if (nevents < 0) {
            printf("kevent error.\n");  // 监听出错
        } else if (nevents > 0) {
            printf("get events number: %d\n", nevents);
            for (int i = 0; i < nevents; ++i) {
                printf("loop index: %d\n", i);
                struct kevent event = event_list[i]; //监听事件的event数据结构
                int clientfd = (int) event.ident;  // 监听描述符
                // 表示该监听描述符出错
                if (event.flags & EV_ERROR) {
                    close(clientfd);
                    printf("EV_ERROR: %s\n", strerror(event_list[i].data));
                }
                // 表示sock有新的连接
                if (clientfd == sock) {
                    printf("new connection\n");
                    struct sockaddr_in client_addr;
                    socklen_t client_addr_len = sizeof(client_addr);
                    int new_fd = accept(sock, (struct sockaddr *) &client_addr, &client_addr_len);
                    char remote[INET_ADDRSTRLEN];
                    printf("connected with ip: %s, port: %d\n",
                           inet_ntop(AF_INET, &client_addr.sin_addr, remote, INET_ADDRSTRLEN),
                           ntohs(client_addr.sin_port));
                }
                if (clientfd == fileno(stdin)) {
                    memset(buf, 0, BUF_SIZE);
                    fgets(buf, BUF_SIZE, stdin);
                    printf("data from stdin: %s\n", buf);
                }
            }
        }
    }
    close(sock);
    return 0;
}
```

## Reference

- [kqueue tutorial](https://wiki.netbsd.org/tutorials/kqueue_tutorial/)
