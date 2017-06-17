title: sed awk学习笔记
date: 2015-11-21 23:59:07
tags: Shell
---



本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

> 由于最近在知乎实习过中不断从大量日志中查找bug的经历, 我决定重新深入学习Shell命令, `以前缺少应用场景导致一些命令并无用武之地`.本文主要针对`sed和awk`做一个简单的学习笔记, 方便以后回顾和再总结


# sed

> `Sed`其实是一种非交互式的编辑器

sed命令格式: `sed [options] {sed-commands} {input-file}`


<!--more-->

**sed处理流程:**

> sed中`sed-commands`执行是有先后顺序的

1. 读入一行内行到缓存空间(模式空间)
2. 从sed-commands中取出一条命令, 判断是否匹配pattern(一般为正则表达式)
3. 如果不匹配, 则忽略后续命令, 回到第二步取下一条命令
4. 如果匹配, 则对缓存的行执行后续的编辑命令; 完成后, 回到第二步继续取下一条命令
5. 当所有命令应用后, 输出缓存行的内容, 回到第一步读取下一行
6. 当所有行处理完则执行结束


```
# 测试文件text.txt. 内容如下(取自参考链接的博文中)
John Daggett, 341 King Road, Plymouth MA
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury MA
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston MA
```


```
# -e表示附加多条编辑命令, 类似与grep中的-e参数

# 's/MA/Andrew/' 将MA替换成Andrew, s表示替换
$ sed -e 's/MA/Andrew/' text.txt
# 执行两条编辑命令
$ sed -e 's/MA/Andrew/' -e 's/PA/, Liu/' text.txt
# 在命令中加p, 表示只输出被编辑的行(但同时有默认的所有行输出)
$ sed 's/MA/Andrewliu/p' text.txt

# -n表示取消默认输出,p表示打印行
$ sed -n 's/MA/Andrewliu/p' text.txt
# 只打印第三行
$sed -n '3p' /etc/passwd
# 打印1，3行
$sed -n '1,3p' /etc/passwd

# d为删除命令, 不指定行号则删除所有行
$ sed 'd' text.txt
# 删除第一行
$ sed '1d' text.txt
# 删除最后一行, $ 表示最后一行
$ sed '$d' text.txt
# 删除包含MA的行
$ sed '/MA/d' text.txt
```

**sed基础命令**

|名称   |命令   |语法|  说明|
|---|---|---|---|
|替换   |s  |`[address]s/pattern/replacement/flags` |替换匹配的内容|
|删除   |d  |`[address]d`|  删除匹配的行|
|插入   |i  |`[line-address]i\text` |在匹配行的前方插入文本|
|追加   |a  |`[line-address]a\text`|    在匹配行的后方插入文本|
|行替换 |c| `[address]c\text`|  将匹配的行替换成文本text|
|打印行 |p  |`[address]p`   |打印在模式空间中的行|
|打印行号|  =   |`[address]=`|  打印当前行行号|
|打印行 |l| `l  [address]l`|    打印在模式空间中的行，同时显示控制字符|
|转换字符|  y   |`[address]y/SET1/SET2/`|   将SET1中出现的字符替换成SET2中对应位置的字符|
|读取下一行 |n| `n  [address]n` |   将下一行的内容读取到模式空间|
|读文件 |r  |`[line-address]r file`|    将指定的文件读取到匹配行之后|
|写文件 |w| `w  [address]w file`|将匹配地址的所有行输出到指定的文件中|
|退出|  q|  `   [line-address]q`    |读取到匹配的行后即推出|

> 注意i/a/c的文本内容都要另起一行

```
$ sed '2a\  
quote> ---------------------------
quote> ' text.txt
```

# awk

> awk是强大的文本分析工具, grep用于查找, sed用于编辑, awk则可以用于文本分析. `awk把文件逐行读入, 以空格为默认分割符将每行切分, 然后进行分析`

```
# 命令格式
$ awk '{pattern + action}' filename
```

PS: `pattern`表示匹配这个模式的行才会执行命令

- `-F`选项用来指定分隔符, 默认使用空格或Tab


简单使用

```
# 只打印所有信息的第一项, $0表示所有域, $1表示第一域, 默认分隔符为空格或Tab
$ ls -la | awk '{print $1}'
# 以:为分隔符, 打印第一列数据, -F ':'等价与`-F:`, 指定多个分隔符`-F '[;:]'`
$ cat /etc/passwd |awk  -F ':'  '{print $1}'
# 格式化输出
$ cat /etc/passwd |awk  -F ':'  '{printf "%s ---- %s\n", $1, $2}'
# 匹配包含root的行
$ cat /etc/passwd |awk  -F:  '/root/'
```

- `print`和`printf`为awk提供的两种打印函数, 前者为直接打印, 后者类似C语言中的格式化输出
- awk更是一种编程语言

```
# 使用逻辑判断过滤数据, 要求第三列数据大于230
cat /etc/passwd |awk  -F ':'  '$3 > 230 {printf "%s : %s\n", $1,$3}'
```

**awk一些内建变量**

|变量名|含义|
|---|---|
|$0|整行内容|
|$1~$n|通过分隔符分割的第n列字段|
|ARGC      |         命令行参数个数|
|ARGV      |         命令行参数排列|
|ENVIRON   |         支持队列中系统环境变量的使用|
|FILENAME  |         awk当前输入文件名|
|FNR       |         浏览文件的记录数, 不同文件各自的行号|
|FS        |         设置输入域分隔符，等价于`-F`选项|
|NF        |         当前记录的字段数, 也就是有多少列|
|NR        |         已读的记录数, 也就是行号|
|OFS       |         输出域分隔符, 默认为空格|
|ORS       |         输出记录分隔符(一行), 默认为换行符|
|RS        |         输入记录分隔符, 默认为换行符|


```
# 输出符合数据的行号和文件名
$ awk  -F ':'  '$3 > 230 {printf "line: %s, filename: %s, %s : %s\n", NR, FILENAME, $1,$3}' /etc/passwd
```


**BEGIN{command}用于指定在开始前执行命令, END{command}指定在对所有行操作结束后执行命令**


```
# 先打印一行name, shell, 然后直接打印$1, $7列, 最后执行END中语句
cat /etc/passwd |awk  -F ':'  'BEGIN {print "name,shell"}  {print $1","$7} END {print "andrew,/bin/zsh"}'
```

> 更复杂的直接用awk写脚本就暂时不涉及了

# 参考链接


- [Sed and awk 笔记之 sed 篇：基础命令](http://kodango.com/sed-and-awk-notes-part-3)
- [AWK 简明教程](http://coolshell.cn/articles/9070.html)
- [linux awk命令详解](http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
- [awk 数据流处理工具](http://linuxtools-rst.readthedocs.org/zh_CN/latest/base/03_text_processing.html#awk)
- [sek和awk](http://dongweiming.github.io/sed_and_awk/#/18)
