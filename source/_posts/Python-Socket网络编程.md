title: Python爬虫(四)--Socket网络编程
date: 2014-12-10 09:23:24
tags: Python
---


---



python的网络变成比c语言简单许多, 封装许多底层的实现细节, 方便程序员使用的同时, 也使程序员比较难了解一些底层的东西, 我觉得学`网络编程`还是用c语言更好一点.


> 写这篇博文, 也希望回顾并整理一下以前学过的c语言和linux下一些东西, 会将一些Linux网络编程的函数和Python网络变成函数做一个简单的对照, 方便记忆


#1. Socket套接字的概念
---

`Socket(翻译为套接字, 我觉得很挫)`,是操作系统内核中的一个数据结构，它是`网络中的节点进行相互通信的门户`。它是网络进程的ID。网络通信，归根到底还是进程间的通信（不同计算机上的进程间通信, 又称进程间通信, IP协议进行的主要是端到端通信）。在网络中，每一个节点（计算机或路由）都有一个网络地址，也就是IP地址。两个进程通信时，首先要确定各自所在的网络节点的网络地址。但是，网络地址只能确定进程所在的计算机，而一台计算机上很可能同时运行着多个进程，所以仅凭网络地址还不能确定到底是和网络中的哪一个进程进行通信，因此套接口中还需要包括其他的信息，也就是端口号（PORT）。在一台计算机中，一个端口号一次只能分配给一个进程，也就是说，在一台计算机中，端口号和进程之间是一一对应关系。
所以，使用端口号和网络地址的组合可以唯一的确定整个网络中的一个网络进程. 


<!--more-->


> 端口号的范围从0~65535，一类是由互联网指派名字和号码公司ICANN负责分配给一些常用的应用程序固定使用的“周知的端口”，其值一般为0~1023, 用户自定义端口号一般大于等于1024, 我比较喜欢用8888

每一个socket都用一个半相关描述{协议、本地地址、本地端口}来表示；一个完整的套接字则用一个相关描述{协议、本地地址、本地端口、远程地址、远程端口}来表示。socket也有一个类似于打开文件的函数调用，该函数返回一个整型的socket描述符，随后的连接建立、数据传输等操作都是通过socket来实现的

##1.1. Socket类型

> socket类型在Liunx和Python是一样的, 只是Python中的类型都定义在`socket模块`中, 调用方式`socket.SOCK_XXXX`

- 流式socket（SOCK_STREAM） `用于TCP通信`

> 流式套接字提供可靠的、面向连接的通信流；它使用TCP协议，从而保证了数据传输的正确性和顺序性

- 数据报socket（SOCK_DGRAM）    `用于UDP通信`

> 数据报套接字定义了一种无连接的服务，数据通过相互独立的报文进行传输，是无序的，并且不保证是可靠、无差错的。它使用数据报协议UDP

- 原始socket（SOCK_RAW）    `用于新的网络协议实现的测试等`

> 原始套接字，普通的套接字无法处理ICMP、IGMP等网络报文，而SOCK_RAW可以, 其次，SOCK_RAW也可以处理特殊的IPv4报文；此外，利用原始套接字，可以通过IP_HDRINCL套接字选项由用户构造IP头。



#2. Socket编程
---

##2.1. TCP通信

TCP通信的基本步骤如下：
服务端：socket---bind---listen---while(True){---accept---recv---send----}---close
客户端：socket----------------------------------connect---send---recv-------close


![TCP](http://picturebag.qiniudn.com/48.png)

**socket函数**
使用给定的地址族、套接字类型、协议编号（默认为0）来创建套接字

```python
#Linux
int socket(int domain, int type, int protocol);
domain：AF_INET：Ipv4网络协议 AF_INET6：IPv6网络协议
type : tcp：SOCK_STREAM   udp：SOCK_DGRAM
protocol : 指定socket所使用的传输协议编号。通常为0.
返回值：成功则返回套接口描述符，失败返回-1。

#python
socket.socket([family[, type[, proto]]])
family : AF_INET (默认ipv4), AF_INET6(ipv6) or AF_UNIX(Unix系统进程间通信). 
type : SOCK_STREAM (TCP), SOCK_DGRAM(UDP) . 
protocol : 一般为0或者默认

如果socket创建失败会抛出一个socket.error异常
```

###2.1.1. 服务器端函数

**bind函数**
将套接字绑定到地址, python下,以元组（host,port）的形式表示地址, Linux下使用`sockaddr_in`结构体指针

```python
#Linux
int bind(int sockfd, struct sockaddr * my_addr, int addrlen);
sockfd : 前面socket()的返回值
my_addr : 结构体指针变量
#####
struct sockaddr_in  //常用的结构体
{
unsigned short int sin_family;  //即为sa_family AF_INET
uint16_t sin_port;  //为使用的port编号
struct in_addr sin_addr;  //为IP地址
unsigned char sin_zero[8];  //未使用
};
struct in_addr
{
uint32_t s_addr;
};
####
addrlen : sockaddr的结构体长度。通常是计算sizeof(struct sockaddr);
返回值：成功则返回0，失败返回-1



#python
s.bind(address)
s为socket.socket()返回的套接字对象
address为元组（host,port）
host: ip地址, 为一个字符串
post: 自定义主机号, 为整型
```

**listen函数**
使服务器的这个端口和IP处于监听状态，等待网络中某一客户机的连接请求。如果客户端有连接请求，端口就会接受这个连接

```python
#Linux
int listen(int sockfd,int backlog);
sockfd : 为前面socket的返回值.
backlog : 指定同时能处理的最大连接要求，通常为10或者5。最大值可设至128
返回值：成功则返回0，失败返回-1

#python
s.listen(backlog)
s为socket.socket()返回的套接字对象
backlog : 操作系统可以挂起的最大连接数量。该值至少为1，大部分应用程序设为5就可以了
```


**accept函数**
接受远程计算机的连接请求，建立起与客户机之间的通信连接。服务器处于监听状态时，如果某时刻获得客户机的连接请求，此时并不是立即处理这个请求，而是将这个请求放在等待队列中，当系统空闲时再处理客户机的连接请求。

```python
#Linux
int accept(int s,struct sockaddr * addr,int * addrlen);
sockfd : 为前面socket的返回值.
addr : 为结构体指针变量，和bind的结构体是同种类型的，系统会把远程主机的信息（远程主机的地址和端口号信息）保存到这个指针所指的结构体中。
addrlen : 表示结构体的长度，为整型指针 
返回值：成功则返回新的socket处理代码new_fd，失败返回-1

#python
s.accept()
s为socket.socket()返回的套接字对象
返回（conn,address）,其中conn是新的套接字对象，可以用来接收和发送数据。address是连接客户端的地址
```

###2.1.2. 客户端函数

**connect函数**
用来请求连接远程服务器


```python
#Linux
int connect (int sockfd,struct sockaddr * serv_addr,int addrlen);
sockfd : 为前面socket的返回值.
serv_addr : 为结构体指针变量，存储着远程服务器的IP与端口号信息
addrlen : 表示结构体变量的长度
返回值：成功则返回0，失败返回-1

#python
s.connect(address)
s为socket.socket()返回的套接字对象
address : 格式为元组（hostname,port），如果连接出错，返回socket.error错误
```


###2.1.3. 通用函数
接收远端主机传来的数据


**recv函数**

```python
#Linux
int recv(int sockfd,void *buf,int len,unsigned int flags);
sockfd : 为前面accept的返回值.也就是新的套接字。
buf : 表示缓冲区
len : 表示缓冲区的长度
flags : 通常为0
返回值：成功则返回实际接收到的字符数，可能会少于你所指定的接收长度。失败返回-1


#python
s.recv(bufsize[,flag])
s为socket.socket()返回的套接字对象
bufsize : 指定要接收的数据大小
flag : 提供有关消息的其他信息，通常可以忽略
返回值为数据以字符串形式
```

**send函数**
发送数据给指定的远端主机

```python
#Linux
int send(int s,const void * msg,int len,unsigned int flags);
sockfd : 为前面socket的返回值.
msg : 一般为常量字符串
len : 表示长度
flags : 通常为0
返回值：成功则返回实际传送出去的字符数，可能会少于你所指定的发送长度。失败返回-1

#python
s.send(string[,flag])
s为socket.socket()返回的套接字对象
string : 要发送的字符串数据 
flag : 提供有关消息的其他信息，通常可以忽略
返回值是要发送的字节数量，该数量可能小于string的字节大小。
s.sendall(string[,flag])
#完整发送TCP数据。将string中的数据发送到连接的套接字，但在返回之前会尝试发送所有数据。
返回值 : 成功返回None，失败则抛出异常。
```

**close函数**
关闭套接字


```python
#Linux
int close(int fd);
fd : 为前面的sockfd
返回值：若文件顺利关闭则返回0，发生错误时返回-1

#python
s.close()
s为socket.socket()返回的套接字对象
```

##2.2. 简单的客户端服务器TCP连接

> 一个简单的回显服务器和客户端模型, 客户端发出的数据, 服务器会回显到客户端的终端上(只是一个简单的模型, 没考虑错误处理等问题)

```python
#服务器端
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import socket   #socket模块
import commands   #执行系统命令模块


BUF_SIZE = 1024  #设置缓冲区大小
server_addr = ('127.0.0.1', 8888)  #IP和端口构成表示地址
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  #生成一个新的socket对象
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  #设置地址复用
server.bind(server_addr)  #绑定地址
server.listen(5)  #监听, 最大监听数为5
while True:
    client, client_addr = server.accept()  #接收TCP连接, 并返回新的套接字和地址
    print 'Connected by', client_addr
    while True :
        data = client.recv(BUF_SIZE)  #从客户端接收数据
        print data
        client.sendall(data)  #发送数据到客户端
server.close()
```


```python
#客户端
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import socket

BUF_SIZE = 1024  #设置缓冲区的大小
server_addr = ('127.0.0.1', 8888)  #IP和端口构成表示地址
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  #返回新的socket对象
client.connect(server_addr)  #要连接的服务器地址
while True:
    data = raw_input("Please input some string > ")  
    client.sendall(data)  #发送数据到服务器
    data = client.recv(BUF_SIZE)  #从服务器端接收数据
    print data
client.close()
```

###2.2.1. 带错误处理的客户端服务器TCP连接

> 在进行网络编程时, 最好使用大量的错误处理, 能够尽量的发现错误, 也能够使代码显得更加严谨

```python
#服务器端
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import sys
import socket   #socket模块

BUF_SIZE = 1024  #设置缓冲区大小
server_addr = ('127.0.0.1', 8888)  #IP和端口构成表示地址
try :
  server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  #生成一个新的socket对象
except socket.error, msg :
    print "Creating Socket Failure. Error Code : " + str(msg[0]) + " Message : " + msg[1]
    sys.exit()
print "Socket Created!"
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  #设置地址复用
try : 
    server.bind(server_addr)  #绑定地址
except socket.error, msg :
  print "Binding Failure. Error Code : " + str(msg[0]) + " Message : " + msg[1]
  sys.exit()
print "Socket Bind!"
server.listen(5)  #监听, 最大监听数为5
print "Socket listening"
while True:
    client, client_addr = server.accept()  #接收TCP连接, 并返回新的套接字和地址, 阻塞函数
    print 'Connected by', client_addr
    while True :
        data = client.recv(BUF_SIZE)  #从客户端接收数据
        print data
        client.sendall(data)  #发送数据到客户端
server.close()
```

```python
#客户端
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import sys
import socket


BUF_SIZE = 1024  #设置缓冲区的大小
server_addr = ('127.0.0.1', 8888)  #IP和端口构成表示地址
try : 
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  #返回新的socket对象
except socket.error, msg :
    print "Creating Socket Failure. Error Code : " + str(msg[0]) + " Message : " + msg[1]
    sys.exit()
client.connect(server_addr)  #要连接的服务器地址
while True:
    data = raw_input("Please input some string > ")  
    if not data :
        print "input can't empty, Please input again.."
        continue
    client.sendall(data)  #发送数据到服务器
    data = client.recv(BUF_SIZE)  #从服务器端接收数据
    print data
client.close()
```


##2.3. UDP通信

UDP通信流程图如下：
服务端：socket---bind---recvfrom---sendto---close
客户端：socket----------sendto---recvfrom---close


![UDP](http://picturebag.qiniudn.com/49.png)


**sendto()函数**
发送UDP数据, 将数据发送到套接字


```python
#Linux
int sendto(int sockfd, const void *msg,int len,unsigned int flags,const struct sockaddr *to, int tolen);
sockfd : 为前面socket的返回值.
msg : 一般为常量字符串
len : 表示长度
flags : 通常为0
to : 表示目地机的IP地址和端口号信息, 表示地址的结构体
tolen : 常常被赋值为sizeof (struct sockaddr)
返回值 :　返回实际发送的数据字节长度或在出现发送错误时返回-1。


#Python
s.sendto(string[,flag],address)
s为socket.socket()返回的套接字对象
address : 指定远程地址, 形式为（ipaddr，port）的元组
flag : 提供有关消息的其他信息，通常可以忽略
返回值 : 发送的字节数。
```


**recvfrom()函数**
接受UDP套接字的数据, 与recv()类似

```python
#Linux
int recvfrom(int sockfd,void *buf,int len,unsigned int flags,struct sockaddr *from,int *fromlen);
sockfd : 为前面socket的返回值.
msg : 一般为常量字符串
len : 表示长度
flags : 通常为0
from :是一个struct sockaddr类型的变量，该变量保存连接机的IP地址及端口号
fromlen : 常置为sizeof (struct sockaddr)。
返回值 : 返回接收到的字节数或当出现错误时返回-1，并置相应的errno。

#Python
s.recvfrom(bufsize[.flag])
返回值 : （data,address）元组, 其中data是包含接收数据的字符串，address是发送数据的套接字地址
bufsize : 指定要接收的数据大小
flag : 提供有关消息的其他信息，通常可以忽略
```


##2.4. 简单的客户端服务器UDP连接

```python
#服务器端
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import socket

BUF_SIZE = 1024  #设置缓冲区大小
server_addr = ('127.0.0.1', 8888)  #IP和端口构成表示地址
server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)  #生成新的套接字对象
server.bind(server_addr)  #套接字绑定IP和端口
while True :
    print "waitting for data"
    data, client_addr = server.recvfrom(BUF_SIZE)  #从客户端接收数据
    print 'Connected by', client_addr, ' Receive Data : ', data
    server.sendto(data, client_addr)  #发送数据给客户端
server.close()
```

```python
#客户端
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import socket
import struct

BUF_SIZE = 1024  #设置缓冲区
server_addr = ('127.0.0.1', 8888)  #IP和端口构成表示地址
client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)  #生成新的套接字对象

while True :
    data = raw_input('Please Input data > ')
    client.sendto(data, server_addr)  #向服务器发送数据
    data, addr = client.recvfrom(BUF_SIZE)  #从服务器接收数据
    print "Data : ", data
client.close()

```




##2.5. 其他


```
s.getpeername()
#返回连接套接字的远程地址。返回值通常是元组（ipaddr,port）。

s.getsockname()
#返回套接字自己的地址。通常是一个元组(ipaddr,port)

s.setsockopt(level,optname,value)
#设置给定套接字选项的值。

s.getsockopt(level,optname[.buflen])
#返回套接字选项的值。

s.settimeout(timeout)
#设置套接字操作的超时期，timeout是一个浮点数，单位是秒。值为None表示没有超时期。一般，超时期应该在刚创建套接字时设置，因为它们可能用于连接的操作（如connect()）

s.gettimeout()
#返回当前超时期的值，单位是秒，如果没有设置超时期，则返回None。

s.fileno()
#返回套接字的文件描述符。

s.setblocking(flag)
#如果flag为0，则将套接字设为非阻塞模式，否则将套接字设为阻塞模式（默认值）。非阻塞模式下，如果调用recv()没有发现任何数据，或send()调用无法立即发送数据，那么将引起socket.error异常。

s.makefile()
#创建一个与该套接字相关连的文件
```


#3. 参考链接
---

[python-socket官方文档](https://docs.python.org/2/library/socket.html)
