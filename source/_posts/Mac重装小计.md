---
layout: post
title: Mac重装小计
date: 2017-06-18 07:00:00
tags: Mac
comments: true
---

<div class="tip">
    本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循<a href="http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh" target="_blank">署名-非商业用途-保持一致</a>的创作共用协议.
</div>

> 很久没有更新博客了, 前段时间迷上了王者农药, 戒了农药后又重新入了暴雪爸爸`暗黑3`的大坑, 呵呵.
> 除此之外, 公司离职回来一直准备毕业论文、毕业答辩、毕业相关材料, 真是焦头烂额.
> 不过想想十九年求学生涯就要结束了, 即将AFK了, 简直幸福.

为什么要重装? Mac系统的乱七八糟的东西已经占据了90%的磁盘空间, 无法减少文件保持一定空闲磁盘空间, 这种情况已经严重影响了我的日常工作.

重装方案严格按照 Apple 官方文档 [如何重新安装 macOS](https://support.apple.com/zh-cn/HT204904) 执行.

## 系统偏好设置

- 允许安装任何来源的APP: `安全性与隐私->通用`. 若无该选项，请命令行执行`sudo spctl --master-disable`
- 设置快捷键: `键盘->快捷键` 更改输入法切换快捷键
- 设置触摸板: 选取全部触摸板设置
- 设置触发角: `桌面与屏幕保护程序->触发角`
- 设置密码验证: `系统偏好设置->安全性与隐私->选择 进入休眠或屏保后立即要求输入密码`

- 编辑 /etc/paths(`sudo vim /etc/paths`)

```c
/usr/local/bin
/usr/local/sbin
/usr/bin
/usr/sbin
/bin
/sbin
```

<!-- more -->

## 软件安装

> 若不需要Xcode可直接跳过该步骤, 安装Homebrew时同样会自动安装`Command Line Tools`

- 通过App Store 安装Xcode
- 安装Command Line Tools

```c
$ xcode-select --install
```

### 安装[Homebrew](http://brew.sh/)

```c
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### 安装Git

```c
$$ brew install git

# SSH-KeyGen, 设置SSH密钥
$ ssh-keygen -t rsa -C "your_email@youremail.com" 

# 在Github中添加新生成的公钥, 验证是否成功请执行以下命令
$ ssh -T git@github.com
```

- 参考[Install-SSH-Use-Github](/2014/09/09/2014-09-09-Install-SSH-Use-Github/)

### 安装nodejs

```c
$ brew install nodejs

# 安装nvm或者n作为nodejs版本控制工具
$ (sudo) npm install -g n
# 如果不行, 则使用nvm进行版本控制
$ n 4.2.4
```

- [设置淘宝npm镜像](https://npm.taobao.org/)

### 安装 Ruby

```c
$ brew install ruby

$ rbenv install -l     # list all available versions
$ rbenv install 2.2.1  # install a Ruby version
$ rbenv global 2.2.1   # set the global version
$ rbenv versions       # list all installed Ruby versions
```

- [配置 gem 墙内源](https://gems.ruby-china.org/)

```c
$ gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
$ gem sources -l
https://gems.ruby-china.org
# 确保只有 gems.ruby-china.org
```

### 安装vim和MacVim

```
Step 1. Install homebrew from here: http://brew.sh
Step 1.1. Run export PATH=/usr/local/bin:$PATH
Step 2. Run brew update
Step 3. Run brew install vim && brew install macvim
Step 4. Run brew link macvim
```

- [Vim配置](https://github.com/humiaozuzu/dot-vimrc)
- [vim.bundle](https://github.com/VundleVim/Vundle.vim)
- [Mac开发利器之程序员编辑器MacVim学习总结](http://blog.csdn.net/eric_xjj/article/details/8932502)

### 安装zsh和[Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh)

```c
$ brew install zsh
# 安装oh-my-zsh
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

- 更换oh my zsh主题在`~/.oh-my-zsh/themes/`路径下, 在[zshthem](http://zshthem.es/)网站进行主题预览
- 安装[autojump](https://github.com/wting/autojump), 一键跳转到目的目录, 不再不停的cd
- 安装[trash](https://github.com/sindresorhus/trash), 不再用rm命令.

```
# 安装autojump
$ brew install autojump
# 安装trash
$ npm install --global trash
```

持久化SSH连接, 安装[mosh](https://mosh.mit.edu/#usage)

```
$ brew install mobile-shell
```

### 安装python

```c
$ brew install python

Pip and setuptools have been installed. To update them
  pip install --upgrade pip setuptools

You can install Python packages with
  pip install <package>

They will install into the site-package directory
  /usr/local/lib/python2.7/site-packages

See: http://docs.brew.sh/Homebrew-and-Python.html
```

### 安装安装 MongoDB, MySQL

```c
$ brew install mongodb mysql
```

设置开机自启动「可选」：

```c
$ mkdir -p ~/Library/LaunchAgents
$ ln -sfv /usr/local/opt/mongodb/*.plist ~/Library/LaunchAgents
$ ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents
```

### .zshrc文件配置

```
alias zshconfig="vim ~/.zshrc"
alias rezsh="source ~/.zshrc"
alias ohmyzsh="cd ~/.oh-my-zsh"

# The default command paramters
alias vi='vim'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias bc='bc -l'
alias wget='wget -c'
alias chown='chown --preserve-root'
alias chgrp='chgrp --preserve-root'
alias rm='rm -I --preserve-root'
alias ln='ln -i'

# Colorful grep output
alias grep='grep --color=auto'
export GREP_COLOR='1;33'

# Colorful ls
export LSCOLORS='Gxfxcxdxdxegedabagacad'
ls='ls --color=auto'

# autojump
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
```

### 安装Sublime Text

1. 安装[Sublime Text 3](http://www.sublimetext.com/)
2. 安装[Package Control](https://packagecontrol.io/)
3. 自定义配置[Settings](https://gist.github.com/Andrew-liu/86223a7f34a17080b0c33e36a09e5da3)

Sublime Text 3 安装Package Control的程序

```
import urllib.request,os;pf = 'Package Control.sublime-package';ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) );open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

## 常用收费软件

- Alfred 2(效率神器)
- Dash 3(程序员专用-文档查询)
- CleanMyMac 3(电脑清理软件)
- PopClip(选中即复制)
- Near Lock(靠近解锁软件)
- Bartender 2(状态栏图标管理器)
- Manico(更方便的软件切换软件)

## 常用免费软件

- Caffeine(不息屏神器)
- XtraFinder(增强Finder)
- Manico(快速切换神器, 相当与window上Alt+Tab)
- Clipy, 重装时发现ClipMenu不维护了, 找了个代替品(剪切板管理)
- Bear，MacDown，Mou(Markdown编辑器)
- Movist(播放器)
- Snip(截图软件)
- Chrome(神之浏览器)
- iTerm2(Mac上遗失的Terminal)
- 搜狗拼音(比原生的好用)
- Window Tidy(窗口大小切换)
- SmoothMouse
- Reeder(RSS订阅)
- [SCROLL REVERSER](http://pilotmoon.com/scrollreverser/) (修改鼠标移动方向, 但不改变触摸板)

- 百度网盘

- 微信
- QQ
- 阿里旺旺
- JetBrain全家桶
- TinyCal
- Shadowsocks
- Spark
- 网易云，QQ音乐
- TickTick
- 富途牛牛
- EverNote
- TeamViewer
- MacTex

## 命令行管理Wifi

[Managing WIFI connections using the Mac OSX terminal command line](http://blog.mattcrampton.com/post/64144666914/managing-wifi-connections-using-the-mac-osx)

## 参考链接

- [Mac开发配置](http://www.jianshu.com/p/77a4349bf67b)
- [shell入门](http://blog.jobbole.com/85702/)
- [what is path of jdk on mac](http://stackoverflow.com/questions/18144660/what-is-path-of-jdk-on-mac)
- [Alfred web search custom](http://alfredtips.com/s/popular/1/)
- [Dash官方文档](https://kapeli.com/dash_guide)
