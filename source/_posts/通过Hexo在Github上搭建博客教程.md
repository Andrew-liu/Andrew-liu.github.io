title: 通过Hexo在Github上搭建博客教程
date: 2014-11-21 13:13:17
tags: [JavaScript, Blog]
---

#Hexo简介

[hexo](http://hexo.io/)是一个基于Node.js的静态博客程序，可以方便的生成静态网页托管在github和Heroku上。作者是来自台湾的`@tommy351`。

> A fast, simple & powerful blog framework, powered by Node.js.

为什么我要使用Hexo:
1. 之前使用过CSDN,博客园和jekyll博客,其中前两个不能算是这个博客吧,主要是使用别的提供的平台发博文,第三个jekyll,我记得是Ruby语言写的,是Github Page着力推荐的, 但其中有繁琐的git操作, 而且有一些资源加载的问题(主要是我不太懂Ruby)
2. 最近在学习JS后端Node.js, 现在火的不行, 异步IO的机制, 所以在学习过程中发现了`Hexo`
3. `Hexo`更加简单优雅, 而且风格多变, 适合程序员搭建个人博客,而且支持多平台的搭建

<!--more-->

> 使用的软硬件

```
Mac Pro    #软件的安装基本都是基于brew
Node.js    #v0.10.33
npm        #2.1.6 包管理,类似于Python中的pip
```

下面正式讲述如何部署:

#1. Git安装和Github设置
使用Mac电脑可以直[brew](http://brew.sh/)安装, 

```
brew install git         #Mac电脑使用brew安装
sudo apt-get install git #Ubuntu系统使用这条命令安装
```

[git操作和github上SSH设置请看这篇博文](http://andrewliu.tk/2014/09/09/2014-09-09-Install-SSH-Use-Github/)

使用Github Page搭建博客, 需要遵循一定的规则, 需要在github建立仓库,仓库名为`Github用户.github.io`, 更多详情请参考[官方文档](https://pages.github.com/)

#2. Node.js安装
mac电脑可以直接通过[brew](http://brew.sh/)安装

```
#安装命令
brew install node  #如果我没记错的话,最新版的node.js的包中已经集成了npm包管理工具

使用以下命令验证是否安装成功
node -v
npm -v
```

Node.js的详细内容请参考[Node.js学习笔记](http://andrewliu.tk/2014/11/02/2014-11-02-Node.js%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)

#3. Hexo安装与设置

Node, npm和Git都安装成功, 开始安装`hexo`,

```
npm install hexo -g  #-g表示全局安装, npm默认为当前项目安装
```

Hexo使用命令:

```
hexo init <folder>  #执行init命令初始化hexo到你指定的目录
hexo generate       #自动根据当前目录下文件,生成静态网页
hexo server         #运行本地服务, 
```

浏览器输入[http://localhost:4000](http://localhost:4000)就可以看到效果。

#4. 添加博文

```
hexo new "postName"  #新建博文,其中postName是博文题目
```
博文会自动生成在博客目录下`source/_posts\postName.md`

```
#文件自动生成格式:
title: "It Starts with iGaze: Visual Attention Driven Networkingwith Smart Glasses"  #博文题目
date: 2014-11-21 11:25:38      #生成时间
tags: Paper                    #标签, 多个标签使用格式[Paper1, Paper2, Paper3,...]
---
```

如果不想博文在首页全部显示, 并能出现`阅读全文`按钮效果, 需要在你想在首页显示的部分下添加`<!--more-->`

```
此处及以上的内容会在首页显示
<!--more-->
一下是在首页隐藏的部分
```

#5. 主题更改

[Hexo](http://hexo.io/)提供了官网的主题, 初始化hexo时也会自动生成一个主题, Hexo还支持个性定制主题, 可以根据自己的喜好对主题进行修改, [更多主题](https://github.com/hexojs/hexo/wiki/Themes)可以在官网中找到

- 个性化博客的设置
在博客的根目录下对喜爱的主题进行主题进行克隆

```
git clone git@github.com:yunlzheng/hexo-themes.git themes/writing

#在./_config.yml，修改主题为writing
theme: writing

#查看本地效果
hexo g       #hexo generate简写
hexo s       #hexo server简写
```


#6. 部署到Github

以上内容都是在本地进行查看, 现在将博客部署到github上
打开./_config.yml

```
# Hexo Configuration
## Docs: http://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: Snow Memory    #博客名
subtitle: 雪忆, 如雪般单纯, 冷静思考.  #博客副标题
description:          #网站描述, 用于爬虫抓取的关键词
author: Andrew Liu    #作者名称
email: Liu.bin.coder@gmail.com  #作者邮箱
language: zh-CN       #网页编码,中文

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://andrewliu.tk   #用于绑定域名, 其他的不需要配置
root: /
permalink: :year/:month/:day/:title/
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
permalink_defaults:

# Directory
source_dir: source  
public_dir: public

# Writing, 设置生成博文的默认格式,不修改
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
highlight:
  enable: true
  line_number: true
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Archives
#Archives 默认值为2,修改为1,Archives页面就只会列出标题,而非全文
## 2: Enable pagination
## 1: Disable pagination
## 0: Fully Disable
archive: 1
category: 1
tag: 1

# Server服务器设置, 不修改
## Hexo uses Connect as a server
## You can customize the logger format as defined in
## http://www.senchalabs.org/connect/logger.html
port: 4000
server_ip: localhost
logger: false
logger_format: dev

# Date / Time format 日期格式
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: MMM D YYYY
time_format: H:mm:ss

# Pagination 分页, 设置每页显示多少篇博文
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Disqus  多说评论, 设置多说短号有效
disqus_shortname: andrewliu

# Extensions
## Plugins: https://github.com/hexojs/hexo/wiki/Plugins
## Themes: https://github.com/hexojs/hexo/wiki/Themes
theme: yilia    #主题设置
exclude_generator:

# Deployment  站点部署到github要配置这里, 非常重要
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: github    #部署类型, 本文使用Github
  repository: git@github.com:Andrew-liu/Andrew-liu.github.io.git  #部署的仓库的SSH
  branch: master   #部署分支,一般使用master主分支
plugins: 
- hexo-generator-feed   #此插件用于RSS订阅, 以后有时间再介绍
```


在配置文件中设置号部署设置后,

```
hexo deploy  #进行部署

Initialized empty Git repository in /Users/andrew_liu/Andrew-liu.github.io/.deploy/.git/
[master (root-commit) bb3079b] First commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 placeholder
[info] Clearing .deploy folder...
[info] Copying files from public folder...
[master 6e24e8d] Site updated: 2014-11-21 11:16:23
 172 files changed, 34969 insertions(+)
 create mode 100644 2014/09/08/Hello-World/index.html
 ...
 create mode 100644 "tags/\345\277\203\350\267\257\346\234\255\350\256\260/index.html"
To git@github.com:Andrew-liu/Andrew-liu.github.io.git
 + 11237d0...6e24e8d master -> master (forced update)
Branch master set up to track remote branch master from git@github.com:Andrew-liu/Andrew-liu.github.io.git.
[info] Deploy done: github


#表示部署成功
```

可以使用`Github用户名.github.io`进行访问, 或者设置个性域名


