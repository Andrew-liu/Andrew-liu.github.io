title: Tmux入门指南
date: 2015-07-12 21:33:36
tags: Blog
---




本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


> 写到今天这篇, Termial神器初成, `Zsh + Oh My Zsh + iTerm2 + Tmux`

# Introduction

tmux是linux中一种管理窗口的程序, 不同于iTerm2, 它提供了一个Session随时存储和恢复的功能(Session概念后面会介绍), detach Session(保持Session后台运行)然后重新attach Session

> 常用场景, 在公司Terimal中开了多个标签和文件, 下班回家忽然有了灵感想要继续编写, 使用ssh远程链接公司电脑, 然后发现标签页和文件都要重新打开, 如果使用Tmux, 下班了detach当前Session, 回家ssh远程连接后, attach Session后, 场景恢复又能愉快的继续编程了...


<!--more-->

# Install

```
# 安装Mac OS X下遗失的包管理Homebrew
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

# 安装神器Tmux
$ brew install tmux
```

# Basic

```
# 启动Tmux
$ tmux
# 关闭Tmux
$ ctrl + d 
# 或退出
$ exit
```

> tmux有三个基本概念：会话(Session)，窗口(Window)和面板(Pane). 当你输入tmux后, tmux实际做的事是首先创建一个会话(Session), 然后在这个会话中创建一个窗口, 你可以继续创建多个窗口(Window), 每个窗口初始只包含一个面板, 继续分屏后, 会出现多个面板(Pane) 你在其中看到的终端实际上都属于tmux的某个面板

更进一步讲, **Session可以包含多个Window, 每个Window又可以包含多个Pane**


![tu](http://ww1.sinaimg.cn/large/ab508d3dgw1etu6swls8oj21kw0m24d2.jpg)

---

**基本操作**

所有快捷键的执行方式:

**重要的事情看三遍**
**重要的事情看三遍**
**重要的事情看三遍**

按下`control + b`两个按键组合, 然后松开`control + b`(为了告诉Tmux我要用Tmux的快捷键了), 然后在按快捷键触发各种行为

例如: `C-b ?`的执行过程为按下`control + b`两个按键组合, 然后松开`control + b`, 然后在按'?'键, 会显示所有快捷键的列表



- `C-b ?` 列出所有快捷键, 按q或Esc返回
- `C-b d`   detach当前会话,可暂时返回Shell界面，输入tmux attach能够重新进入之前会话
- `C-b s`   选择并切换会话；在同时开启了多个会话时使用

# Shortcut


**Window操作**

- `C-b c` 创建一个新窗口
- `C-b &` 关闭当前窗口
- `C-b w` 列出所有的窗口选择
- `C-b p` 切换到上一个窗口
- `C-b n` 切换到下一个窗口
- `C-b 窗口号` 使用窗口号切换窗口(例如窗口号为1的, 则`C-b 1`)
- `C-b ,`   重命名当前窗口，便于识别各个窗口


**Pane操作**

- `C-b %` 横向分Terminal
- `C-b "` 纵向分Terminal
- `C-b 方向键` 则会在自由选择各面板
- `C-b x` 关闭当前pane
- `C-b q`   显示面板编号

**Session**

```
# 创建一个新的session
$ tmux new -s <name-of-my-session>

# 在当前session中创建一个新的Session, 并保证之前session依然存在
# C-b : 然后输入下面命令
new -s <name-of-my-new-session>

# 进入名为test的session
$ tmux attach -t test
```


- `C-b s` 列出所有会话
- `C-b d` detach当前session(可以认为后台运行)


# Pro

**美化Tmux**

使用[gpakosz的Tmux配置](https://github.com/gpakosz/.tmux)进行美化

**优点**

- 使用`C-a`作为前缀更方便使用, 同时保存了`C-b`的触发前缀
- powerline状态条美化(用过vim的都应该比较熟悉)
- 显示笔记本电池状态

安装使用

```
$ cd
$ rm -rf .tmux
$ git clone https://github.com/gpakosz/.tmux.git
$ ln -s .tmux/.tmux.conf
$ cp .tmux/.tmux.conf.local .
```

# 参考链接


- [Tmux官网](https://tmux.github.io/)
- [A Tmux crash course: tips and tweaks](http://tangosource.com/blog/a-tmux-crash-course-tips-and-tweaks/)
- [tmux入门指南](http://abyssly.com/2013/11/04/tmux_intro/)
- [Tmux - Linux从业者必备利器](http://cenalulu.github.io/linux/tmux/)
- [Tmux conf](https://github.com/gpakosz/.tmux)
- [Linux下终端利器tmux](http://kumu-linux.github.io/blog/2013/08/06/tmux/)

