title: 试玩USRP
date: 2015-02-08 14:39:21
tags: USRP
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


#1. USRP基本构成
---

USRP(`Universal Software Radio Peripheral`, 通用软件无线电外设)

PC可以使用USRP作为射频前端, USRP内部做一些数字基带处理和一系列中频处理, USRP上层使用开源的`GNURadio`.

<!--more-->

- 支持USB2.0(理论上达到32MB/s)
- FPGA(子板将载波信号模拟下变频到模拟中频后, A/D转换器将模拟中频转换为数字中频, DDC(digital down converter在FPGA内部)将数据中频降到数字基带, 同时, 对数字信号执行抽取操作, 是的数据可以被USB和PC处理)
- 4个高速A/D转换器(12-bit), 每个转换器采样速率64M/s
- 4个高速D/A转换器(14-bit), 每个转换器时钟频率128MB/s
- 4个插槽, 分别为RXA, TXA, RXB, TXB. 可以同时插两个单收子板和两个单发子板, 或者同时插2个收发子板(`我使用了RFX子板`, RF前端是实现在子板上的, 不同的子板处理不同的频率带宽)

#2. USRP启动

USRP本身不含ROM, 当插入到PC的USB口后, PC需要下载固件到USB控制芯片上.
[USRP固件主页](http://uhd.ettus.com)
[USRP固件下载地址](https://github.com/EttusResearch/uhd)

> 首先我不推荐在Mac上安装固件和GnuRadio, 绝对是大坑, 尤其是对10.10.2最新版本的用户, 差点把我坑死

ubuntu安装方式:

```
sudo bash -c 'echo "deb http://files.ettus.com/binaries/uhd/repo/uhd/ubuntu/`lsb_release -cs` `lsb_release -cs` main" > /etc/apt/sources.list.d/ettus.list'
sudo apt-get update
sudo apt-get install -t `lsb_release -cs` uhd
```

由于我执行以上命令出现子进程错误, 所以我决定直接对源代码编译执行

```
$ git clone git://github.com/EttusResearch/uhd.git
$ sudo apt-get install libboost-all-dev libusb-1.0-0-dev python-cheetah doxygen python-docutils cmake
$ cd <uhd-repo-path>/host #默认路径是uhd/host
$ mkdir build  #较早的uhd版本本身存在build文件夹
$ cd build
$ cmake ../
$ make
$ make test
$ sudo make install
$ sudo ldconfig
#下面两个命令主要是进行测试
$ sudo uhd_find_devices  
$ sudo uhd_usrp_probe

```

> 测试成功则表示uhd固件安装成功, 可以识别USRP了


> 这个环境和固件问题还是比较多的, 在Mac直接无法安装成功, 现在ubuntu上虽然可以, 但是也是很麻烦的

#3. GNU Radio

[源代码下载](https://github.com/gnuradio/gnuradio)

使用源码安装会有一些奇怪的错误,
```
$ mkdir build
$ cd build
$ cmake ../
$ make && make test
$ sudo make install
$ sudo ldconfig  
$ gnuradio-config-info -v  #测试GnuRadio的版本
```

如果使用了ubuntu的话, 直接使用一下命令

```
$ sudo apt-get install gnuradio
```

> 我安装的版本`uhd_release_003_005`_000和`gnuradio3.7.2.1`, 试了很多才才找到整个兼容的搭配, 如果uhd版本和gnuradio发生core dump错误


#4. 参考链接
---


[GnuRadio安装流程](https://gnuradio.org/redmine/projects/gnuradio/wiki/InstallingGR#Linux)
[Installation procedure for UHD 003.005.002 and GNU Radio v3.6.4.1 on Ubuntu v12.04](http://www.orbit-lab.org/wiki/Documentation/dSDR/GNURadio/InstallationOnUbuntu12.04)
[git tag切换](http://git-scm.com/book/zh/v1/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE)

