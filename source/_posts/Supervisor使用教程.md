title: Supervisor使用教程
date: 2016-02-19 23:50:05
tags: Blog
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

[Supervisor: 一个进程控制系统](http://supervisord.org/), 用来监控类UNIX系统中的进程, 能够方便的启动, 重启, 关闭进程(`已经尝试过Python和Scala项目`)

## 安装

Supervisor使用Python开发, 必然是可以使用`pip`进行安装

```
$ (sudo) pip install supervisor
```

<!--more-->

## 配置

通过`echo_supervisord_conf`命令将配置重定向到配置文件中

```
$ echo_supervisord_conf > /etc/supervisord.conf  
```

自动生成的配置如下:

```
[unix_http_server]
file=/tmp/supervisor.sock   ; UNIX socket 文件，supervisorctl 会使用
;chmod=0700                 ; socket 文件的 mode，默认是 0700
;chown=nobody:nogroup       ; socket 文件的 owner，格式： uid:gid

;[inet_http_server]         ; HTTP 服务器，提供 web 管理界面
;port=127.0.0.1:9001        ; Web 管理后台运行的 IP 和端口，如果开放到公网，需要注意安全性
;username=user              ; 登录管理后台的用户名
;password=123               ; 登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log ; 日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ; 日志文件大小，超出会 rotate，默认 50MB
logfile_backups=10           ; 日志文件保留备份数量默认 10
loglevel=info                ; 日志级别，默认 info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ; pid 文件
nodaemon=false               ; 是否在前台启动，默认是 false，即以 daemon 的方式启动
minfds=1024                  ; 可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ; 可以打开的进程数的最小值，默认 200

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; 通过 UNIX socket 连接 supervisord，路径与 unix_http_server 部分的 file 一致
;serverurl=http://127.0.0.1:9001 ; 通过 HTTP 的方式连接 supervisord

; 包含其他的配置文件
[include]
files = relative/directory/*.ini    ; 可以是 *.conf 或 *.ini
```

- 通过命令运行supervisord

```
# 或者其他任意路径下的配置文件
$ supervisord -c /etc/supervisord.conf
```


## 启动进程

我们需要对自己的服务进行一些配置. 服务管理一般将配置文件存放在`/data/etc/supervisor/conf.d`路径下.

- `[program:x]`, 其中x为进程名, 必不可少的
- `command`, 项目要运行的命令, 必不可少的
- `process_name`, 进程名, 如果要启动多个进程, 则修改修改, 默认为`%(program_name)%`
- `numprocs`, 启动多个项目实例

```
# 文件名为 some-project.conf
[program:some-project]  # program后跟着进程名是必须的
command = /data/apps/some-project/bin/python /data/apps/doraemon/some-project/main.py
autostart = true
autorestart = true  # 服务挂掉会自动重启
loglevel = info  # 输出日志级别
stdout_logfile = /data/log/supervisor/some-project-stdout.log
stderr_logfile = /data/log/supervisor/some-project-stderr.log
stdout_logfile_maxbytes = 500MB
stdout_logfile_backups = 50
stdout_capture_maxbytes = 1MB
stdout_events_enabled = false
```

配置的模板基本如上, 只需要根据自己的服务定制`command`就可以了.

```
# scala项目的配置
[program:owl-collector-scala]
command=java -jar /data/apps/owl-collector-scala/target/scala-2.11/owl-collector-scala-assembly-1.0.jar
autostart=false
autorestart=true  
stdout_logfile=/data/log/supervisor/owl-collector-scala.stdout.log
stderr_logfile=/data/log/supervisor/owl-collector-scala.stderr.log
stdout_logfile_maxbytes=500MB
stdout_logfile_backups=50
stdout_capture_maxbytes=1MB
stdout_events_enabled=false
loglevel=info
priority=1100
```

配置完成后, 进行`supervisorctl`命令行管理shell, 输入`reload`会进行重新加载进程配置.

```
# 加载配置
$ supervisorctl
supervisor> reload
Really restart the remote supervisord process y/N? y
Restarted supervisord
# 查看当前进程状态
supervisor> status
some-project          STOPPED   Feb 19 11:33 PM
# 启动进程
supervisor> start some-project
```

## supervisorctl

`supervisorctl`是一个命令行工具. 可以与不同的supervisord进程进行通信, 获取子进程信息, 管理子进程.


```
# shell中输入supervisorctl, 进入交互式的界面
$ supervisorctl
> status  # 查看当前supervisor管理的进程状态和运行时间
> start some-project  # 启动some-project进程
> stop some-project  # 关闭some-project进程
> restart some-project  # 重启some-project进程
> reload  # 重新加载配置文件(当增加新的配置文件时执行)

> tail -f some-project stderr  # 实时查看some-project的错误日志
```

> 不怎么使用Web进行进程管理, 更详细的配置信息可以查看`官方文档`

## 参考链接

- [supervisor官方文档](http://supervisord.org/running.html)
- [How to automatically start supervisord on Linux (Ubuntu)](http://serverfault.com/questions/96499/how-to-automatically-start-supervisord-on-linux-ubuntu)
- [进程的守护神 - Supervisor](http://linbo.github.io/2013/04/04/supervisor/)
