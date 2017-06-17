title: Mac下配置安卓开发环境(Eclipse)
date: 2015-06-20 09:24:57
tags: Android
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


> 由于Mac下intellij莫名其妙的原因(反正至今找不到原因), 一创建Android项目就卡死, 没有办法, 只能继续在Eclipse下折腾了

> 记住我们的口号: 生命不止, 折腾不息

<!--more-->

#Java JDK下载

[Java JDK(Java SE Development Kit)官方下载](http://www.oracle.com/technetwork/cn/java/javase/downloads/index.html) 

- 按照平台下载安装

**Mac获得Java的安装位置**

```
$ echo $(/usr/libexec/java_home)  #输出当前JDK路径

安装的JDK8: /Library/Java/JavaVirtualMachines/jdk1.8.0_20.jdk
Mac自带的JDK6: /System/Library/Java/JavaVirtualMachines/1.6.0.jdk
```

#Android SDK下载

谷歌系列的东西需要出(fan)墙(qiang), 在`Other Tools Only`下载自己平台的SDK

- [Android SDK官方下载](https://developer.android.com/sdk/index.html)
- 无法翻墙的请使用国内资源进行下载[Android Dev Tools](http://www.androiddevtools.cn/)

下载完, `自行解压`下载好的Android SDK，打开Eclipse —> Preferences对话框，找到Android那一项，单击Browse，选取SDK目录即可。

#Eclipse安装

- [Eclipse安装](https://eclipse.org/downloads/) 

**下载对应平台, 然后安装就可以了, 没有什么好说的**



#ADT安装

*ADT(Android Developer Tools)*

打开Eclipse，在菜单中选择Help中的`Install New Software`, 在右上方选择Add

- Name编辑为`ADT`
- Location编辑为`https://dl-ssl.google.com/android/eclipse/ `(**你懂的要翻**)

![ADT](http://ww2.sinaimg.cn/large/ab508d3dgw1et6u5z5ixtj21ei10iai5.jpg)

选择`Developer Tools`, 点击`next`执行下载



**安装平台工具**

安装好重启后, 在`Window`中`Android SDK Manager`中下载`Tools`和`Extras`以及其他API

![API](http://ww1.sinaimg.cn/large/ab508d3dgw1et6u97zwf2j21640yeaoq.jpg)


#虚拟机设置

在Window中打开`Android Virtual Device Manager`, Create添加一个Android虚拟机, 用于开发的Android程序做测试.

![虚拟机](http://ww3.sinaimg.cn/large/ab508d3dgw1et6uciepjoj21320rejwh.jpg)


#创建Android工程


重启Eclipse程序后, File->New->Other->Android Application Project. 设置项目名等条目, 创建成功

![创建](http://ww3.sinaimg.cn/large/ab508d3dgw1et6uf08ddjj20t80qyjwn.jpg)

![设置](http://ww4.sinaimg.cn/large/ab508d3dgw1et6ufgyg32j20ym0sm7bw.jpg)安装成功


#FAQ


[AppCompat v7 r21 returning error in values.xml?](http://stackoverflow.com/questions/26457096/appcompat-v7-r21-returning-error-in-values-xml)


搭建过程中会出现各种各样的问题, 遇到问题时, 大家最好把出错原因放到Google中搜索, 最后总能解决的, 实在解决不了, 可以找了安卓开发者帮你看看, 最后祝大家都能成功配置安卓开发环境.

