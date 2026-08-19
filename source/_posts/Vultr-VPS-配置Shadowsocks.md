title: Vultr VPS 配置Shadowsocks
date: 2015-08-29 22:47:18
tags: Blog
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

![google](http://ww4.sinaimg.cn/large/ab508d3dgw1euxk2uxlogj20uk0fywfy.jpg)


> 曲径悲剧, [红杏](http://honx.in/i/VAJbS-z5NHn0VT_3)感觉好日子也快到头了, 最近明显感觉访问Google速度下降了不少(延迟也高了), 于是终于决定自己折腾另谋出路.. 连电信都要推出氮气瓶业务了, 我们也该自己学着搭VPS了, 然后我学会的时候, Shadowsocks被干掉了, 真是一个搞笑的故事, 不过现在依然能用, 只是不维护了...

<!--more-->

# 云服务选择

测试了几个云服务提供商, 包括[Digital Ocean](https://www.digitalocean.com/), [搬瓦工](https://bandwagonhost.com/), [Vultr](https://www.vultr.com/), 前两个大概都在200ms左右, Digital Ocean丢包较少, 搬瓦工价格便宜配置低, Vultr的优势在于有日本机房(`大概40-100ms左右, 偶有丢包`), 配置比Digital Ocean稍微好一点, 像比较牛的Linode, 我等屌丝玩不起, 就不研(zhuang)究(bi)了...


> 最后, 我觉得先试用下**Vultr**

通过[两月免费使用, 赠送50刀](http://www.vultr.com/freetrial?ref=6835439), 填写信用卡后, 会扣费$2.5(测试账户用), 之后会返还给你.


言归正传, 记录自己搭VPS的过程...

# 账号注册

- 使用试用通道, 填写信用卡信息, 邮箱密码信息
- 邮箱验证


注册成功后, 你可以在Billing(账单)标签界面看到赠送的50$, 以及当前信用卡信息.

![credit](http://ww2.sinaimg.cn/large/ab508d3dgw1euxk2cmav9j210g0syq8f.jpg)

既然有钱了, 我们可以开始折腾了...

# 服务选择


在Deploy(部署)页面自行选择希望使用的服务器配置, 操作系统, 配置需要的价格等等情况.

- 硬盘最好选SSD(没有原因, 就是快)
- Operating System(操作系统)选择Centos 6 x64(64位系统)
- Location(服务器位置)选择Tokyo(就是为了东京机房(离国内近, 速度快), 我们才选择Vultr的服务呀)
- Server Size(服务器配置)我选择了768内存的最便宜的服务, 一个月5$
- 选中Enable Private Network
- Server Label(服务器别名)随便起个好(zhuang)听(bi)的就行
- Place Order(然后会自动部署服务器)


![配置1](http://ww1.sinaimg.cn/large/ab508d3dgw1euxk3ckse7j21f40v2gug.jpg)
![配置2](http://ww3.sinaimg.cn/large/ab508d3dgw1euxk3spw5qj21fe0zon80.jpg)

> 此时, 部署成功后, 会收到一封有你服务器ip地址的, 现在可以ssh连接服务器, 开始瞎折腾了..

# 环境配置


- 在My Servers标签中可以查看自己当前部署服务器的状态, 看到Running表示服务器正在运行, 等待你朝他开刀...
- 点击Manage查看root账户初始密码, 然后使用ip, user和password就可以愉快的登陆VPS了

![密码](http://ww2.sinaimg.cn/large/ab508d3dgw1euxk4y2ocpj213g0yojy0.jpg)


**使用自己电脑打开Terminal**, 输入下面命令

```
# ip address为当前Server的ip地址
$ ssh root@ip address
# 输入初始密码
```
登陆成功显示如下:

![登陆](http://ww3.sinaimg.cn/large/ab508d3dgw1euxk45pnj9j20ro026q3i.jpg)

- 马上更改密码

```
$ passwd
# 输入新密码两次
```

- 安装各种配置工具

```
# 一些安装包需要的编译工具
$ yum install gcc
$ yum install libevent
$ yum install python-devel

# 安装python的pip包管理
$ yum install python-setuptools && easy_install pip
$ pip install gevent

# 安装加密工具
$ yum install openssl-devel
$ yum install swig
$ pip install M2Crypto

# 安装shadowsocks服务器端
$ pip install shadowsocks
```

# 服务器配置

1. 自定义配置文件

```
# 使用vi新建shadowsocks.json配置文件
$ vi  /etc/shadowsocks.json
```

直接输入`:set paste`, 然后输入`i`进入Paste insert状态

```
# 粘贴下列数据到配置文件中
{
    "server":"ip_address",
    "server_port":9999,
    "local_port":1080,
    "password":"password",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false,
    "workers": 1
}
```

> 此处`ip_address`为在Vultr上部署VPS的ip号, `server_port`可以可以任意选择大于1023的端口号(尽量取的大一些). `password`为连接服务器的密码, 其他的无序更改.

完成后, 按`esc`退出编辑模式. 输入`:wq（有冒号）`以保存并退出


```
# 命令行后台启动服务
$ nohup ssserver -c /etc/shadowsocks.json &
```

到此为止, 服务器端配置完成...

# 客户端配置


到[Shadowsocks官方网站](http://shadowsocks.org/en/download/clients.html)下载客户端(PC/Android/iOS都有).

不同的操作系统下载不同的客户端软件.


如图选择服务器->打开服务器设置->左下角选择+(加号)

![加号](http://ww4.sinaimg.cn/large/ab508d3dgw1euxk65h41mj20lg0hodjq.jpg)

填写新的服务器配置.

![服务](http://ww4.sinaimg.cn/large/ab508d3dgw1euxk5m9rs3j20sa0he767.jpg)

添加在Vultr上配置的服务器的ip, 自己在shadowsocks中设置的端口号和密码.

> 打开Shadowsocks, PC/Android/iOS可以翻墙google了...

完成...
