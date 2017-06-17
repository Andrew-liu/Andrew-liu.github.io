title: Buildout使用小结
date: 2015-11-07 15:40:31
tags: Python
---
本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

## buildout简介


`Buildout`是一个基于Python的构建工具, `Buildout`主要是为了解决两个问题:

- 中心化的应用组装和部署
- 重复的从Python软件发布中组装项目

通过一个配置文件`buildout.cfg`, 可以从多个部分创建、组装并部署你的应用, 能够构建一个封闭隔离的开发环境.

<!--more-->


## buildout安装

```
pip install zc.buildout
```


## buildout使用

- 创建一个项目目录:

```
# 目录名可以随便起
$ mkdir buildout
$ cd buildout
```

- 初始化项目目录

```
$ buildout init
```

查看buildout后的目录结构

```
$ tree ./

├── bin
│   └── buildout
├── buildout.cfg
├── develop-eggs
├── eggs
│   ├── setuptools-18.4-py2.7.egg
│   └── zc.buildout-2.4.6-py2.7.egg
└── parts
```


配置buildout.cfg文件 

```
[buildout]  # 脚本入口
show-picked-versions = true # 显示所安装的版本
parts = app  # 相当于入口执行的子函数, 可以设置多个parts

[app]  # 编写子函数app的逻辑
recipe = zc.recipe.egg  # 除了recipe其他都是选项都被认为是recipe的参数
eggs = 
    pymongo  #需要安装的依赖
interpreter = python  # 设置要安装的解释器

```

保存后, 然后执行

```
$ buildout
```

buildout的流程, 先调用`[buildout]`, 然后发现parts中有app这个子函数, 然后调用app这个子函数的逻辑, app中除了recipe, 其他都被认为是recipe的桉树, 当调用eggs时, buildout发现这些包没有被安装, 于是自动安装包并存放在`eggs目录下`

- buildout会在eggs目录下安装zc.buildout, pymongo
- 在bin目录下生成一系列可执行文件, 此时我们如果想解释任何python脚本文件, 都必须执行`bin/python xxx.py`(即当前buildout的bin目录中的python解释器)
- 每个可执行文件中的sys路径都发生改变, 都会优先读取eggs下的三方包


## buildout结合setup.py

将setup.py中填写的name项对应的值, 填写到eggs中, 则在buildout会自动加载setup.py中的配置

创建`setup.py`文件

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from setuptools import setup, find_packages


setup(
    name='test',  # 此处填写包名
    version='0.0.1',
    author='andrewliu',
    author_email='liu.bin.coder@gmail.com',
    description='This is just a test',
    license='PRIVATE',
    keyword='test',
    packages=find_packages('apps'),
    install_requires=[
        'nose',  # 此处填写需要的包
        'pymongo',
        'mysql-python',
        'redis',
    ],
)
```


并修改buildout.cfg文件

```
[buildout]
develop = .
show-picked-versions = true
parts = app

[app]
recipe = zc.recipe.egg
eggs =
    test
interpreter = python
```

再次执行buildout, 会解析setup.py的数据, 并安装`install_requires`中填写的所有包, 并且会通过` packages=find_packages`将当前目录加入到`sys.path`


```
# 执行buildout可得到下面输出. 会安装buildout和setup.py中的所有包
[versions]
MySQL-python = 1.2.5
setuptools = 18.5
zc.buildout = 2.4.7
zc.recipe.egg = 2.0.3

# Required by:
# test==0.0.1
nose = 1.3.7

# Required by:
# test==0.0.1
pymongo = 3.1

# Required by:
# test==0.0.1
redis = 2.10.5
```
**可以看到setup.py中的需求包全被安装了!!!**

> 验证有效性

```
# 创建test文件, 添加代码
$ vim test_path.py

#!/usr/bin/env python
# -*- coding: utf-8 -*-

import sys

if __name__ == '__main__':
    print(sys.path)
```

现在我们需要使用当前目录bin下的可执行文件python来运行代码

```
$ bin/python test_path.py
```

**运行结果如下**

```
['/Users/andrew_liu/Zhihu/buildout/buildout', '/Users/andrew_liu/Zhihu/buildout/buildout/eggs/redis-2.10.5-py2.7.egg', '/Users/andrew_liu/Zhihu/buildout/buildout/eggs/MySQL_python-1.2.5-py2.7-macosx-10.10-intel.egg', '/Users/andrew_liu/Zhihu/buildout/buildout/eggs/pymongo-3.1-py2.7-macosx-10.9-intel.egg', '/Users/andrew_liu/Zhihu/buildout/buildout/eggs/nose-1.3.7-py2.7.egg', '/Users/andrew_liu/Zhihu/buildout/buildout/bin', '/Library/Python/2.7/site-packages/distribute-0.6.49-py2.7.egg', '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python27.zip', '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7', '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/plat-darwin', '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/plat-mac', '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/plat-mac/lib-scriptpackages', '/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python', '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/lib-tk', '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/lib-old', '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/lib-dynload', '/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/PyObjC', '/Library/Python/2.7/site-packages']
```

- 首先会搜索当前目录下的包
- 然后会搜索eggs下由buildout安装的包, `可以看出buildout的作用: 优先加载当前目录eggs下的包`
- 最后`才会搜索系统中的包`. 可以看到并没有完全隔离系统中的包. 


## 生成完全隔离的开发环境

使用工具:

- setuptools
- zc.buildout
- virtualenv

**创建一个完全隔离的开发环境**

1. 创建一个空的项目文件
2. 使用virtualenv创建一个虚拟环境
3. 使用buildout来配置开发的需求.
4. 结果setup.py集成测试, 开发, 分布于一体.

**终**



## 参考链接

- [Buildout 入门](http://www.aireadfun.com/blog/2013/08/29/buildout-ru-men/)
- [用Buildout来构建Python项目](https://lxneng.com/posts/192)
- [buildout使用小例](http://www.worldhello.net/2010/12/10/2207.html)
