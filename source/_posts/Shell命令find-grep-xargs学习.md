title: Shell命令find grep xargs学习
date: 2015-07-31 10:14:16
tags: [Linux, Shell]
---


本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


> 这篇博文这两天写好了, 由于最近比较忙, 所以提前发了, 希望Deadline不要延期.

#find命令

find命令用于在一个目录(及子目录)中搜索文件, 可以指定匹配条件, 如文件名, 文件类型

> 在Mac下有强大的Spotlight和Alfred(感觉window下的everything更牛叉), 所以find用的就比较少了

```
find [-H | -L | -P] [-EXdsx] [-f path] path ... [expression]
```

<!--more-->

常用形式为:

```
# path表示find命令查找的目录路径
$ find [path] [expression]

# 查找当前路径下的所有git为前缀的文件, 这里*被称为通配符
find . -name 'git*'

# 查找当前路径下所有以.txt和.pdf为后缀的文件
$ find . \( -name "*.txt" -o -name "*.pdf" \) -print


# !表示否则, 查找所有不是以.txt为后缀的文件
$ find . ! -name "*.txt" -print

# 按类型搜索, 只所有目录(f表示文件 l表示符号链接(软链接) d表示目录)
$ find . -type d -print 

# 按用户搜索  找到所有andrew_liu的用户文件
$ find . -type f -user andrew_liu -print

# 按照时间搜索 -atime -30m  搜索30分钟内被访问的文件(后面有详细解释)
$ find . -atime -30m -type f -print
# -atime +30m  搜索超过30分钟被访问的文件
$ find . -atime +30m -type f -print

# 找到以.txt为后缀的文件后删除
$ find . -type f -name "*.txt" -delete
```


**按时间搜索**(这些元数据都在inode的结构体中有记录)

- atime 访问时间 (单位有一周w, 一天d, 一小时h, 一分钟m, 一秒s, 以下类似）
- mtime 修改时间 （内容被修改）
- ctime 变化时间 （元数据或权限变化）


#grep命令

> grep命令是强大的`文本搜索命令`

- grep全称是`globally search a regular expression and print`, 表示全局`正则表达式`匹配并输出, 它的使用权限是`所有用户`
- 存在很多grep的修改版, 例如agrep表示`近似的grep`approximate grep用于模糊字符串搜索, fgrep用于`固定样式搜索`fixed pattern searches, 而egrep用于`搜索更复杂的正则表达式`语法(摘自wiki)



> **使用格式**: grep [options] match_patten file

`option`这里值列举几个常用的选项, 其他可以使用`man grep`进行查看

- `-c`:只输出匹配行的次数
- `-l`:查询多文件时只输出包含匹配字符的文件名
- `-n`:显示匹配行及行号
- `-v`:显示不匹配行
- `-i`:搜索时忽略大小写
- `-l`:只打印包含匹配行的文件名
- `-e`:指明一个查找模式(常用多一次匹配多个查找模式)
- `-R`:递归的查找多级目录


```
# 匹配test.doc文件中所有存在hello的行
$ grep 'hello' test.doc

# 匹配test.doc文件中所有存在hello的行, 并打印行号
$ grep -n 'hello' test.doc

# 找到所有行不匹配hel或者how, -e匹配多个模式
$ grep -v -e 'hel' -e 'how' test.doc

# 在多级目录中对文本递归搜索
$ grep "class" . -R -n
#输出结果:
./test/class.c:1:class
./test/class.c:2:class yes
./test/class.c:4:class fuck
./test.doc:5:class
./test.pdf:1:class

# 找到所有以.hel开头的行, 并使用管道命令
$ cat test.doc | grep '^\.hel'

# 找到所有包含hel行的行数(只输出行数, 没有内容)
cat test.doc | grep -c 'hel'

# 更多使用选项使用
$ man grep
```

关于正则表达式的学习可以看[Python正则表达式](http://andrewliu.tk/2014/10/26/2014-10-26-Python%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/)


# xargs命令

- xargs 能够将输入数据转化为特定命令的命令行参数,可以配合很多命令来组合使用
- 经常和find, grep通过管道连接使用

xargs参数说明

- `-d` 定义定界符 （默认为空格 多行的定界符为 n）
- `-n` 指定输出为多行
- `-I` {} 指定替换字符串，这个字符串在xargs扩展时会被替换掉,用于- `待执行的命令需要多个参数时
- `-0`：指定0为输入定界符

```
# 将多行输出转化为单行输出
$ cat test.doc|xargs

# 将单行输出转化为多行输出, -n指定每行显示的单词数, 下面表示将单行转换为每行显示五个单词
$ cat new.pdf| xargs -n 5
Here you can select if
you want to set the
gesture for Magic Mouse or
for the Trackpad or for
the Keyboard
```


# 其他常用小命令

```
# 清空文件
$ :> a.txt

# 将文本中的制表符转换为空格
$ cat text| tr '\t' ' '  //制表符转空格

# 统计行数, 常用统计整个项目的代码量
$ wc -l file 

# 打包当前文件夹(不压缩), -c为打包选项, -v为显示打包进度, -f为使用档案文件
$ tar -cvf test.tar ./

# 解包 -x为解包选项
$ tar -xvf test.tar

# 压缩文件, 生成文件为.gz后缀
$ gzip test.tar

# 解压缩文件, 解压为test.tar
$ gzip test.tar.gz

# 查看端口占用的进程状态
$ lsof -i:5000

# 端口号被占用:
$ sudo netstat -tulpn | grep 80
$ sudo netstat -aWn --programs | grep 80

# 查询被监听的端口
$ lsof -i tcp | grep LISTEN

# 通过PID进程号杀死进程
$ sudo kill -s (PID)

# DNS查询，寻找域名domain对应的IP
$ host andrewliu.tk
andrewliu.tk has address 103.245.222.133

# SSH登陆:
$ ssh username@host

# ftp/sftp文件传输(重要)
$ sftp username@host

# 远程文件复制, scp -r 要复制的整个目录 username@host:目的目录
$ $scp localpath ID@host:path
$ scp -r MonitorTrend bin_liu@10.64.24.91:Flask

# 添加新的用户和密码
$ useradd -m username
$ passwd username

# 删除用户
$ userdel -r username

# 不同用户之间切换
$ su userB
```


# 参考链接

- [grep](https://zh.wikipedia.org/zh-cn/Grep)
- [Linux Shell Scripting Tutorial (LSST) v2.0](http://bash.cyberciti.biz/guide/Main_Page)
- [Linux Tools Quick Tutorial](http://linuxtools-rst.readthedocs.org/zh_CN/latest/base/01_use_man.html)
- [sar 找出系统瓶颈的利器](http://linuxtools-rst.readthedocs.org/zh_CN/latest/tool/sar.html#sar)


