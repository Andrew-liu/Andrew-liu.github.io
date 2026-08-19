title: Sublime Text 2/3使用心得
date: 2014-11-25 12:56:30
tags: [Mac, Blog]
---

---

> [Sublime Text](http://www.sublimetext.com/)号称`最性感的编辑器, 跨平台, 免费使用`


> PS:本文主要针对Mac下的Sublime Text配置, 其他的请自行对快捷键修改, 之前写错了, Mac下使用的是Sublime Text2, 在另一台电脑用的Sublime Text3, 混肴了, 但在配置方面2/3只有少量的差别, 更多的版本差别体现在插件的支持上


#0. 设置subl命令行

```
#如果是在默认shell下, 
sudo ln -s "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" /usr/bin/subl

#使用zsh的可以使用以下命令
alias subl="'/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl'"
alias nano="subl"
export EDITOR="subl"
```

测试使用一下命令
```
$ subl
```

使用方法
```
用法:
subl [arguments] [files]         编辑指定的文件edit the given files
   or: subl [arguments] [directories]   打开指定的目录
   or: subl [arguments] -               编辑stdin

参数Arguments:
  --project <project>: 载入指定的project
  --command <command>: 运行指定的命令
  -n or --new-window:  打开一个新的窗口
  -a or --add:         添加文件夹到当前窗口
  -w or --wait:        返回前等待文件关闭
  -b or --background:  不激活该应用程序
  -s or --stay:        文件关闭后保持应用程序激活状态
  -h or --help:        显示帮助并退出
  -v or --version:     显示版本信息并退出

如果从标准输入--wait是隐式的。 使用--stay当文件关闭是不切换到后台控制台(只与是否有等待的文件有关)

文件名可以通过加:line或者:line:column后缀来指定打开的定位。
用法摘自官方文档
```

<!--more-->

#1. 修改Sublime Text2 默认配置
---

在菜单栏选择 Sublime Text->Preferences->Setting-User(注意其中Setting-Default是默认的系统配置, 是不可修改的), 通过修改用户设置会覆盖系统对应的默认配置,下面是我的配置单, 都加油注释

```py
{
    "color_scheme": "Packages/Theme - itg.flat/itg.dark.tmTheme", #主题设置, 这是下载主题后, 自动生成的, 也可以手动配置
    "font_size": 15, #设置字体大小, 我比较喜欢大一点的字体
    "ignored_packages":  #设置忽略文件类型, 第二个是默认忽略的, 第一个markdown文件我使用另一种文件打开,
    [
        "Markdown",
        "Vintage"
    ],
    "create_window_at_startup": false, #取消启动时,自动打开新窗口的设置, 这个设置很恶心, 每次启动后会自动生成一个空白窗口
    "open_files_in_new_window": false, #取消打开文件时会新生成一个窗口, 默认设置每次打开一个项目会重新生成一个窗口
    "highlight_line": true, #高亮当前编辑行, 其实高亮的不明显
    "highlight_modified_tabs": true, #设置文件修改时, 标签高亮提示, 这样可以提示保存
    "show_encoding": true, #在窗口右下角显示打开文件的编码
    "original_color_scheme": "Packages/Theme - itg.flat/itg.dark.tmTheme",  #主题设置
    "translate_tabs_to_spaces": true #将tab键的形式改为四个空格
}
```


#2. 添加快捷键前端网页调试功能
---

> 这个功能是我以前在github的项目里看到的, 已经找不到项目源地址了, 感谢原作者

一、点击菜单Tools -> New Plugin...，在创建好的py文件输入下列内容：

```py
import sublime, sublime_plugin
import webbrowser
 
url_map = {
    '/Users/andrew_liu/HTML/' : 'file:///Users/andrew_liu/HTML/',#这里需要进行个人电脑的配置, 配置个人项目路径
}
 
class OpenBrowserCommand(sublime_plugin.TextCommand):
    def run(self, edit) :
        window = sublime.active_window()
        window.run_command('save')
        url = self.view.file_name()
        flag = False
        for path, domain in url_map.items():
            if url.startswith(path):
                url = url.replace(path, domain).replace('\\', '\/')
                flag = True
                break
        if not flag:
            url = 'file://' + url
        webbrowser.open_new(url) #这里使用默认的浏览器调试
```

将文件保存到Packages/User目录（Packages可通过菜单里的Browser Packages...打开），文件名随意，如open_browser.py。插件部分完工了。


二、接下来，为刚才的插件分配快捷键。点菜单Tools -> Command Palette...，或者`shift+cmd+p`，打开命令集，选择“key Bindings - User”打开个人快捷键配置，输入下列内容：

`[{ "keys": ["ctrl+shift+b"], "command": "open_browser" }]
`
这就是要做的全部工作，可以测试下了。打开一个html文件，ctrl+shift+b试试，没意外的话文件会在默认浏览器打开了。url_map里配置的站点目录到URL的映射应该也是可用的。


#3. 添加包管管理神器
---

最近Package Control好像被墙了, 我的另一台电脑老是上不去, 具体不太清清楚, 天朝丧心病狂大家懂得, 所以如果一直上不去, 请`翻墙`

安装过程: 使用快捷键  **control + `** 或者菜单栏选择`View > Show Console`

- Sublime Text3在控制台输入

```
import urllib.request,os,hashlib; h = '7183a2d3e96f11eeadd761d777e62404' + 'e330c659d4bb41d3bdf022e94cab3cd0'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

- Sublime Text2在控制台输入

```
import urllib2,os,hashlib; h = '7183a2d3e96f11eeadd761d777e62404' + 'e330c659d4bb41d3bdf022e94cab3cd0'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler()) ); by = urllib2.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); open( os.path.join( ipp, pf), 'wb' ).write(by) if dh == h else None; print('Error validating download (got %s instead of %s), please try manual install' % (dh, h) if dh != h else 'Please restart Sublime Text to finish installation')
```

打开包管理神器  请使用快捷键`shift + cmd + p`, 然后输入package或者一些简写


#4. Sublime Text 常用快捷键

|快捷键组合|功能|
|---|---|
|`shift + cmd + p`|打开命令面板|
|**control + `**|控制台|
|`cmd + n`|新建标签|
|`cmd + 数字`|标签切换|
|`cmd + option + 2`|分成两屏|
|`control + 数字`|分屏时移动到不同的屏幕|
|`cmd + delelte`|删除光标前所有字符, 貌似是Mac快捷键|
|`cmd + f`| 查找|
|`option + cmd + f`|查找替换|
|`cmd + t`|文件跳转|
|`control + g`|行跳转, 类似vim中的num + gg|
|`cmd + r`|函数跳转|
|`cmd + /`|给选中行添加或去掉注释|
|`cmd + [或 cmd + ]`|智能行缩进|
|`cmd + k + b`|开关侧边栏|


更多快捷键可查看[官方文档](http://www.sublimetext.com/docs/3/)

#5. 推荐插件
---

> 插件是非常重要的一部分, 一个普通的编辑器难以满足大部分人需要, 更难以满足程序员多样化的编程语言, 所以需要使用插件打造`个性化的类IDE`, 相比与IDE有启动快, 干净, 干扰少的优点

##5.1 主题类:

- 包含大量配色主题的插件包
首先介绍一个包含大量配色包的网站, [Colorsublime](http://colorsublime.com/), 里面各种各样的配色让人眼花缭乱
[Colorsublime Plugin](https://github.com/Colorsublime/Colorsublime-Plugin)

安装方法:
```
shift + cmd + p 打开命令面板
输入 “Package Control: Install Package” 命令
输入 Colorsublime plugin, 找到后回车安装
安装成功后在preferences中选择配色
```
[Colorsublime Plugin github项目地址](https://github.com/Colorsublime/Colorsublime-Plugin)

![Colorsublime](http://picturebag.qiniudn.com/Snip20141125_6.png)

- iTg主题, 我的最爱

安装方法
```
shift + cmd + p 打开命令面板
输入 “Package Control: Install Package” 命令
输入Theme - itg.flat, 找到后回车安装
安装成功后在preferences中选择主题
```

[项目github地址](https://github.com/itsthatguy/theme-itg-flat)

![itg-flag](http://picturebag.qiniudn.com/Snip20141125_5.png)

![itg-flag](http://picturebag.qiniudn.com/Snip20141125_4.png)

- 著名的Soda主题

安装方法

```
shift + cmd + p 打开命令面板
输入 “Package Control: Install Package” 命令
输入soda, 找到Theme-Soda,找到后回车安装
安装成功后在preferences中选择Setting-User更改主题设置:
{
    "theme": "Soda Light 3.sublime-theme"
}
```

[github项目地址](https://github.com/buymeasoda/soda-theme/)


![Soda-Light](http://picturebag.qiniudn.com/Snip20141125_2.png)

![Soda-Dark](http://picturebag.qiniudn.com/Snip20141125_3.png)

##5.2. 其他插件

安装方法都通过Package Control
```
shift + cmd + p 打开命令面板
输入 “Package Control: Install Package” 命令
输入安装插件的简写或全拼,找到后回车安装
```

- alignment
这个忘了干嘛的了, 好像是控制所有类型文本的缩进
- all Autocomplete
sublime只对当前文件进行本文件中的查找不全, `all Autocomplete`是对全部打开的文件进行查找不全, 选择更多更全面
- converttoUTF8
编辑的所有文件都使用UTF-8编码
- docblockr
强大的文档注释功能, 只要在文档中输入`/*`然后按一下tab, 就会根据代码自动生成注释,
- emmet
前端神器, 减少大量的工作量, 使用方法可以参考[Emmet：HTML/CSS代码快速编写神器](http://www.iteye.com/news/27580)或者官方文档
- git
支持sublime上的git操作, 这个就不用多说了
- markdownediting或者markdownPerview
这个是写Markdown必备的。可以在包管理器中安装。装完之后，写作Markdown时（右下角显示语法为Markdown），可以按ctrl+b，直接就会生成HTML，并在浏览器中显示。
- jsformat
JavaScript代码格式化
- sidebarenhancement
这是用来增强左边的侧边栏。左侧边栏可以在View -> Side Bar -> Show Side Bar中打开，可以用Project -> Add Folder to Project...往侧边栏加入常用的文件夹。装完这个插件，侧边栏的右键菜单会多一些功能，挺实用的。
- Bracket Highlighter
这是用来做括号匹配高亮的，可以在包管理器中安装。Sublime Text 2自带的括号匹配只有小小的一横线，太不显眼了，这个可以让高亮变成大大的一坨，不过我觉得它大得会盖住光标了。

- SublimeLinter
语法检测工具, 可以检测到所写代码的语法错误,并高亮显示错误
[用户手册](http://sublimelinter.readthedocs.org/en/latest/about.html)
其中需要额外安装一下包, 如`SublimeLinter-pyflakes and SublimeLinter-pep8.SublimeLinter-jshint, SublimeLinter-pyyaml, SublimeLinter-csslint, SublimeLinter-html-tidy, and SublimeLinter-json`

[更多](https://github.com/SublimeLinter)

- Djaneiro
支持模版和关键词高亮, 提供有用的代码片段


---
未安装:

- Anaconda
[打造python IDE](http://damnwidget.github.io/anaconda/)
> Anaconda turns your Sublime Text 3 in a full featured Python development IDE including autocompletion, code linting, IDE features, autopep8 formating, McCabe complexity checker and Vagrant for Sublime Text 3
