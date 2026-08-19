---
title: macOS Sierra 惊险升级
date: 2016-09-24 18:26:27
tags: Blog
---

## 惊现问题

`2016年9月21` Apple开始推送 `macOS Sierra(10.12)`.

**此处升级的亮点:**

- **亮点就是没有亮点!!!**
- 最大的升级是`Mac OS X` 改名为 `macOS'`, 很大的改变有木有
- Mac增加了Siri支持, 我知道我Mac多了个天气预报小助手
- 可以使用Apple Watch自动近距离解锁Mac, 听说[Near Lock]()已哭晕在厕所? 然而首先你要买一部 `Apple Watch`
- 跨设备复制粘贴, 可以使用云端剪切板, iPhone上复制的东西可以在Mac上直接黏贴. 然而首先你要买一部 `iPhone`
- Safari我就不喷了, 反正用了Chrome的我实在受不了龟速的Safari. 听说Safari很省电, 这个卖点不错!
- 还有啥? 这次升级只有很少的App闪退阵亡.(呵呵

> 然后开始作死升级之路... 怎么升级就不说了, 正常人都知道....

<!--more-->

![](http://ww3.sinaimg.cn/large/ab508d3dgw1f81hwo4xcwj20qo0zk0t7.jpg)

告诉我macOS未能安装在电脑上, 磁盘没有足够的空间来安装..

我当时就哔了狗了, 你安装前不先检查磁盘剩余空间就敢给我安装?
重启后, 自动又开始安装, 然后再一次错误, 我就知道出事了...

都是我最近下片无数, 看完不删还想有空再回味一下, 这下子出事了... 磁盘不够了...


## 尝试解决

方法一. `Failed`. 安全模式方法, 重启后, 按住`Shift`等待出现苹果标志, 进入Mac模式, 听说此模式可以进入电脑, 这样我就能够删除我的大片了, 然后就有足够空间了, 然而我天真了, 真的再也进不去系统了, 一到80%或者100%就卡住不动了,太坑爹了...

![](http://ww3.sinaimg.cn/large/ab508d3dgw1f81i4hw4xaj20qo0zk3yx.jpg)

方法二. `Failed`. 恢复模式方法, 重启后, 按住`Command + R`进入恢复模式, 在菜单栏的工具中, 打开Terminal, 满心以为可以通过Terminal中删除home文件下的文件. 结果我发现里面全是一些系统文件, 没有任何个人文件的影响, 此方法猝..


方法三. `Failed`. Ubuntu大法. 插入制作了Ubuntu的U盘, 按住`Option`开机, 开机后选址`Efi`(大概是这个名字)模式, 然后选择`Try Ubuntu but not installing`方式试用Ubuntu. 通过Ubuntu挂载Mac的硬盘, 然后对Mac硬盘的文件进行删除. 结果我又天真了, 由于硬盘未能安装成功更新, 导致挂载失败, 只能挂载恢复盘的文件, 我要这玩意有何用...

```
# 安装hfsprogs用于支持hfsplus
$ sudo apt-get install hfsprogs

# 测试要挂载盘的状态, sdXX表示要挂载的磁盘, 如sda1, sda2
$ sudo fsck.hfsplus -f /dev/sdXX
# 输出结果为 The volume #### appears to be OK表示可以挂载

# 挂载磁盘到本地的路径下, 此处我挂载sda2到本地/home/ubuntu路径下
sudo mount -t hfsplus -o fore,rw /dev/sda2 /home/ubuntu
# 若挂载成功可进行读写操作
```

![](http://ww3.sinaimg.cn/large/ab508d3dgw1f81ihpzf4cj20qo0zkta2.jpg)

4. `Success`. 最后实在没办法了, 我又让他重启自动安装了, 结果发现这次成功了, 我还能说什么????

![](http://ww4.sinaimg.cn/large/ab508d3dgw1f81ij42nrpj20js08adgg.jpg)

## 自我反省

1. 以后再升级, 千万不能硬上了, 记得备份呀. 说的我好像买的起`Time Machine`一样
2. 升级前一定要看看剩余的磁盘空间呀, 千万别装备呀, 千万别信Apple粑粑的升级包呀


> 最后我只想大声说一句: 重启大法好!!!!
