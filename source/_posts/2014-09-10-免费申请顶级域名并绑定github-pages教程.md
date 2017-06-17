---
layout: post
time: 2014-09-10 10:02:00 +0800
title: 免费申请顶级域名并绑定github-pages教程
tags: Linux
comments: true
---

`本人博文中所有出现的username 请全部改成自己的个人账户或者github id`

#1. 使用jekyll+github配置好自己的个人博客

```
git init
git add .
git commit -m "blog"
git remote add blog git@github.com:username/username.github.io.git
//username更改为自己github账户名  并已经建立了给username.github.io
git push -u blog master
//第一次上传后 ，如果配置争取， 十分钟后就可以使用username.github.io访问自己的个人博客
```

<!--more-->

[Github Pages官方手册](https://pages.github.com/)

#2. 申请免费的顶级域名
[free domains域名免费注册](http://www.dot.tk/en/index.html?lang=en)

1. 申请自己喜欢的域名. 例如 andrewliu ,点击Go
2. 在use your new domian 选择use DNSs
3. `通过自己username.github.io， 获取对应的ip地址，具体步骤如下 ：`
- 打开终端，输入命令 `ping username.github.io`
- 记录下方出现的ip地址， 填写到dns设置中
4. Registration length选择12months
5. 然后注册个人账户
6. 会发一封激活邮件到邮箱， 请注意激活


#3. 绑定顶级域名到github
在本地个人blog的根目录下

```
vim CNAME
//在CNAME中输入刚才注册的域名， 例如我的andrewliu.tk， 注意不要加入www

然后上传至github
git add CNAME
git commit -m "dns"
git push blog master
```
大工造成
   
