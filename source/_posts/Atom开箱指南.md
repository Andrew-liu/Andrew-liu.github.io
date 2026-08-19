title: Atom开箱指南
date: 2015-12-03 22:25:44
tags: Blog
---


本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

## 简介和安装

![大图](http://ww4.sinaimg.cn/large/ab508d3djw1eymt6nor7uj21kw0xzqdr.jpg)

> 狗带一个官方简介: Atom是一个现代文本编辑器, 高度可定制而不需要写烦人的配置文件, `另外Github出品必属精品, 现在应该是社区推动了`, 我是世界最大男性交友网站的脑残粉...

- 自带包管理`apm`
- 智能补全, 预装文件浏览树
- 跨平台编辑(好像我在说废话, 有名的Editor都能做到这些
- 颜控必备, 丰富UI和语法高亮, 还能自己进行配置
- 听说对前端开发很友好

<!--more-->

- 直接[Atom官方](https://atom.io/)下载app.
- 使用 [homebrew-cask](https://github.com/caskroom/homebrew-cask)安装.

```
# 通过brew安装cask
$ brew install caskroom/cask/brew-cask
# 使用cask安装Atom
$ brew cask install atom

```


## 修改设置

> Atom基本是可以开箱即用的, 不过还需要修改一些小设置

- `Preferences`->`Settings`->选取`Soft Tabs`
- `Tab Length: 4`

**即设置Tab使用4个空格**

## 常用快捷键


|快捷键|说明|
|---|---|
|`shift + cmd + p`|命令版(可以看到所有快捷键)|
|`alt + shift + s`|查看文件相关语言的代码块(snippet)|
|`cmd + f`|搜索当前文件|
|`cmd+shift+f`|搜索整个项目|
|`alt + cmd + [`|代码折叠, 我不喜欢用|
|`alt + cmd + ]`|代码展开|
|`cmd + /`|快速注释当前行|
|`cmd + [`|代码左缩进|
|`cmd + ]`|代码右缩进|
|`cmd + b`|快速跳转打开的文件|


|**光标移动**快捷键|说明|
|---|---|
|`alt+B或alt+left`|光标按单词左移|
|`alt+F或alt+right`|光标按单词右移|
|`cmd+right或ctrl+e`|光标移动到行最右最后一个非空字符|
|`cmd+left或ctrl+a`|光标移动到行最左第一个非空字符|
|`cmd + up`|光标移动到文件头|
|`cmd + down`|贯标移动到文件尾|
|`ctrl + g`|行跳转, 语法为行号:列号|
|`cmd + r` |按当前文件方法跳转|
|`cmd + t`|全项目模糊查找关键字并跳转|
|`ctrl + m`|按照括号匹配跳转|

> 书签功能: 通过`cmd + F2`给某一行设置书签, 书签的标志出现在行号右侧, 通过`F2`进行书签跳转. `此功能超赞`

|**选择和编辑快捷键**| 说明|
|---|---|
|`ctrl+shift+p`|向上选中行|
|`ctrl+shift+n`|向下选中行|
|`cmd + a`|选中整个文本|
|`cmd + l`|选中整行|
|`cmd + d`|多重选中, 用过sublime的都很熟悉|
|`ctrl+shit+k`|删除整行|
|`cmd + delete`|删除光标到行首|
|`alt + delete`|按单词删除|




## 推荐插件

> 插件的安装推荐使用`apm`, (不要直接用setting中install装, 会爆炸的... Ps: Python党多装python相关插件

**Atom包管理用法:**

```
# 安装指定包
$ apm install <package_name>
# 安装指定版本的包
$ apm install <package_name>@<package_version>
# 查找包
$ apm search <package_name>
# 查看包更多详情
$ apm view <packge_name>
# 查看当前已安装包(包含默认Atom捆绑和个人安装)
$ apm list
```

**首先祭出个人已安装的package列表,然后一一介绍**

```
/Users/andrew_liu/.atom/packages (9)
├── activate-power-mode@0.3.2
├── autocomplete-python@1.0.1
├── emmet@2.3.16
├── linter@1.11.3
├── linter-flake8@1.9.2
├── linter-pep8@1.0.1
├── python-tools@0.6.7
├── script@3.2.0
└── seti-ui@0.8.1
```

- 代码爆炸效果插件 [activate-power-mode](https://atom.io/packages/activate-power-mode)

```
$ apm install activate-power-mode
```

代码爆炸式, 编辑器也会同时发生晃动, 现在已经有了解决方案[编辑器同时晃动解决方案](https://github.com/JoelBesada/activate-power-mode/issues/63).


效果不多说, [效果链接](https://github.com/JoelBesada/activate-power-mode), 装逼必备, 不过编辑器晃得眼晕


- 代码风格审查 [linter](https://atom.io/packages/linter)

> 注意: 安装linter需要安装相关语言代码的风格审查工具才能生效, [全语言风格审查列表](http://atomlinter.github.io/)

```
$ apm linter
$ (sudo) pip install pep8
$ (sudo) pip install flake8
# 安装插件
$ apm install pep8
$ apm install linter-flake8
```
- 前端神器 [emmet](https://github.com/emmetio/emmet-atom)

```
$ apm install emmet
```

不多说了, [传送门: 使用emmet](http://andrewliu.in/2014/11/25/Sublime-Text-3%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97/#5-2-_其他插件)

- 函数定义跳转 [python-tools](https://atom.io/packages/python-tools)

支持快速变量重命名, 快速函数定义跳转(`ctrl+alt+g`), 选中string文本

- Python自动补全 [autocomplete-python](https://atom.io/packages/autocomplete-python)

```
$ apm install python-tools 
```

官方Atom插件中已经捆绑了`language-python插件`.  不过此插件提供了更强大变量名函数和标准库的补全.

- 脚本运行 script

通过文件名(`cmd + i`), 选中的代码或者行号来运行代码

```
$ apm install script
```

![运行脚本](http://ww4.sinaimg.cn/large/ab508d3djw1eymtu4t7quj20w612gtei.jpg)

- 编程语言图标定制主题 [seti-ui](https://atom.io/themes/seti-ui)

一个针对编程语言文件的图标进行定制的UI主题, 另有搭配的[语法高亮主题](https://atom.io/themes/seti-syntax)

![样式](http://ww3.sinaimg.cn/large/ab508d3djw1eymtzmkbjrj216u10wgsx.jpg)




`Ps`: 没有提到Git相关插件, 因为本人喜欢用命令行git.
> 解锁更多姿势可以查看`Atom官方文档`


## 参考链接

[Atom官方文档直通车](https://atom.io/docs)
[Sublime Text 2/3使用心得](http://andrewliu.in/2014/11/25/Sublime-Text-3%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97/#0-_设置subl命令行)

