---
layout: post
date: 2014-09-09 15:37:00 +0800
title: Install-SSH-Use-Github
tags: github
comments: true
---

#1. github上的操作

##1.1. 初始化操作
在http://github.com 上注册自己的github账户后，要执行语句
git config --global user.name "yourname"
git config --global user.email "XXX@XXX.com"
这样等于是在本地标记了自己的账户，以后就可以直接使用了。

##1.2. 在最后有获取密钥的方法

1. 要上传文件到GitHub的Git系统上，需要一个SSH密匙来认证，在文章最后有方法。
2. 将SSH Key提交到GitHub `在设置中添加`

<!--more-->

##1.3. 基本git操作
上传代码到git的步骤：
1. git init
建立一个仓库。

2. git add XXX
添加文件XXX。add后面加“.”，添加当前目录所有文件。

3. git commit -m 'message' (`最后一个附加信息相当于标签`)
上传时附加说明信息message。

4. git remote add origin XXX@github.com:XXX/XXX.git   (`给git仓库的地址取一个别名`)
这一句将origin用作XXX@github.com:XXX/XXX.git的别名，以后就可以方便的用origin代表XXX@github.com:XXX/XXX.git了。
> git remote rm origin用于删除别名

5. git push -u origin master   上传带`master`标签的文件到github
上传当前目录到github的master分支

> 如果原来仓库中有另外一个文件, 首先应该进行github仓库代码下载, 合并后总体上传,因为github并不能识别代码是不是一个项目


#1.4. git常用命令
1. git clone
eg： git clone git://github.com/XXX/XXX.git some_project  (最后为仓库的地址)
将'git://github.com/XXX/XXX.git'这个URL地址的远程版本库完全克隆到本地XXX目录下面

2. git init
初始化当前目录，初始化后，在当前目录下会出现一个名为 .git 的目录，所有 Git 需要的数据和资源都存放在这个目录中

3. git status
查看当前目录相应文件在git下的状态

4. git commit
提交当前工作空间的修改内容
eg：git commit -m "update code"'，`提交的时候必须用-m来输入一条提交信息`

5. git add
将当前更改或者新增的文件加入到git的索引中
eg：git add test.c 就会增加test.c文件到git的索引中
git add .  ——  将当前所有的更改加入到git索引中

6. git pull
> 意思就是将远程的代码给拉到本地
从其他的版本库(既可以是远程的也可以是本地的)将代码更新到本地
eg：'git pull origin master 就是将origin这个版本库的代码更新到本地的master主枝

7. git rm
从当前的工作空间中和索引中删除文件
eg：git rm test.c

8. git push
将本地commit的代码更新到远程版本库中(推送的远程的代码仓库中)
eg：git push origin 就会将本地的代码更新到名为orgin的远程版本库中

9. git log , git diff , git reset --hard HEAD^

> git log //用于查看提交的每个版本

> git diff //用于查看提交的本版本的改变

        在Git中，用HEAD表示当前版本也就是最新的提交,上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。Git在内部有个指向当前版本的HEAD指针，当你回退版本的时候，Git仅仅是把HEAD从指向另一个提交版本的标签.
> $ git reset --hard HEAD^


----

#2. 修改主机名方法

修改命令行主机名：sudo gedit /etc/hosts
 
 #修改主机名为路径名
修改一下使命令行只显示当前目录的最后一级目录名，这样看起来也好，用pwd可以看到完整的路径名。
   找到配置文件先进行备份： sudo  cp  ~/.bashrc  ~/.bashrc-bak
   找到配置文件修改： sudo  gedit  ~/.bashrc
   下面是我的bashrc，真正修改到就一行代码：

----------

```
# If this is an xterm set the title to user@host:dir
# 配置标题
case "$TERM" in
xterm*|rxvt*)
#    PS1="/[/e]0;${debian_chroot:+($debian_chroot)}/u@/h: /w/a/]$PS1"
PS1="[/u@/h:/W]//$ "   //起头写。只需要修改这一行
    ;;
*)
    ;;
esac
```

保存，重启终端就行了

------

#3. SSH的安装以后获取SSH公钥

1 前提：已安装ssh
 > 安装方法 sudo apt-get install openssh-server
 
2 检查SSH公钥
cd ~/.ssh
看看存不存在.ssh，如果存在的话，掠过下一步；不存在的请看下一步

3 生成SSH公钥

```  
$ ssh-keygen -t rsa -C "your_email@youremail.com" 
# Creates a new ssh key using the provided email Generating public/private rsa key pair. 
Enter file in which to save the key (/home/you/.ssh/id_rsa):
```
按回车，现在你可以看到，在自己的目录下，有一个.ssh目录，说明成功了

3.1 输入github密码
Enter passphrase (empty for no passphrase): [Type a passphrase] 
Enter same passphrase again: [Type passphrase again]
这个时候输入你在github上设置的密码。

3.2 然后在.ssh中可以看到

```  
Your identification has been saved in /home/you/.ssh/id_rsa. 
# Your public key has been saved in /home/you/.ssh/id_rsa.pub.
# The key fingerprint is: 
# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@youremail.com
```
4 添加SSH公钥到github
打开github，找到账户里面添加SSH，把id_rsa.pub内容复制到key里面。

5 测试是否生效
使用下面的命令测试

ssh -T git@github.com
当你看到这些内容放入时候，直接yes

```  
The authenticity of host 'github.com (207.97.227.239)' can't be established. 
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48. 
Are you sure you want to continue connecting (yes/no)?
//看到这个内容放入时候，说明就成功了。

Hi username! 
You've successfully authenticated, but GitHub does not provide shell access.
```


在开源项目中点击fork那个按钮，稍等一会，项目便会拷贝一份到自己的respositories中。那么如何把代码检到本地呢？

查看右方下载地址

```  
$git clone git@github:liveNo/webFile.git  
```

然后输入密码执行完成得到项目。这时，你在本地的检出路径会看到项目。

