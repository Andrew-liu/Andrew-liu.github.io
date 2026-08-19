title: Django搭建简易博客教程(二)-开发环境和Django安装
date: 2014-12-22 10:56:09
tags: [Django, Python]
---

#开发环境
---

`下面仅仅是我的项目开发环境, 没有必要追求完全一致...`


```
Mac OS X 10.10.1
Python3.4.1
Django1.7.1 
Bootstrap3.3.0  #非必要
Sublime Text 3  #非必要
virtualenv  1.11.6
```

<!--more-->

#虚拟环境配置
---

使用`virtualenv`创建虚拟环境, Ubuntun和Mac安装程序基本一致

```
#安装virtualenv
$ pip install virtualenv  
#创建虚拟环境
$ virtualenv -p /usr/local/bin/python3.4 ENV3.4  

Running virtualenv with interpreter /usr/local/bin/python3.4
Using base prefix '/Library/Frameworks/Python.framework/Versions/3.4'
New python executable in ENV3.4/bin/python3.4
Also creating executable in ENV3.4/bin/python
Installing setuptools, pip...done.

#激活虚拟环境
$ source /ENV3.4/bin/activate  
#查看当前环境下的安装包
$ pip list  
pip (1.5.6)
setuptools (3.6)
```

更多`virtualenv`使用可以参考[Virtualenv简明教程](http://andrewliu.tk/2014/12/08/Virtualenv%E7%AE%80%E6%98%8E%E6%95%99%E7%A8%8B/)

#Git安装
---

Git是目前世界上最先进的分布式版本控制系统

**Mac下git安装**

```
$ brew install git
```

**Ubuntu下git安装**

```
$ sudo apt-get install git
```

**Windows**就不说了, 没怎么用过Windows做开发, 坑太多了


##Github创建

在[Github](https://github.com/)中创建一个属于自己的帐号
新建帐号后, 请点击`New repository`或者下图地方

![Github仓库创建](http://picturebag.qiniudn.com/58.png)

并通过[Install-SSH-Use-Github](http://andrewliu.tk/2014/09/09/2014-09-09-Install-SSH-Use-Github/)学习简单的Github与git的协作以及SSH的创建

> Github和git的协作我们会在使用的时候重复提示, 但最好先进行`SSH的安装和配置`


#Django安装
---

安装最新版的Django版本

```
#安装最新版本的Django
$ pip install  django 
#或者指定安装版本
pip install -v django==1.7.1
```

#Bootstrap安装
---

> `Bootstrap` 简洁、直观、强悍的前端开发框架，让web开发更迅速、简单


bootstrap已经有较为完善的中文文档, 可以在[bootstrap中文网](http://v3.bootcss.com/getting-started/#download)查看

推荐下载其中的Bootstrap源码

> 到目前为止, 基本环境已经搭建好了
