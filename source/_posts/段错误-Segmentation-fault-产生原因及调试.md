title: 段错误(Segmentation fault)产生原因及调试
date: 2015-10-25 22:32:46
tags: C++
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


## 段错误

段错误是由`内存管理单元`的`异常`导致的, 这个异常通常是由于解引用一个未初始化的或非法的指针引起的, 此时程序尝试访问的内存位置并不允许程序去访问. 

<!--more-->


## 段错误产生原因

- 访问了不存在的内存地址
- 访问了系统保护的内存地址(解引用一个空指针)
- 向制度的内存地址写入信息
- 用完堆或者栈空间


```
# test.c 访问系统保护的内存地址引发段错误

#include <stdio.h>
#include <stdlib.h>

int main(int argc, const char *argv[]) {
    int* ptr = (int*)0;
    *ptr = 100;
    return 0;
}
```

## 段错误调式方法

使用**gdb**调试程序, 在gcc编译时加入`-g`参数

```
$ gcc -g test.c -o test
```

**Tips**:

```
# gdb安装方法
$ brew install homebrew/dupes/gdb
```

更详细的步骤可以查看[Installing GDB on Mac OS X Yosemite](http://www.patosai.com/blog/post/installing-gdb-on-mac-os-x-yosemite)

```
# 运行一下命令进入gdb调式状态
$ gdb test
(gdb) run
Program received signal SIGSEGV, Segmentation fault.
0x0000000100000f90 in main (argc=1, argv=0x7fff5fbff318) at test.c:6
6       *ptr = 100;
```

可以达到触发了段错误, 并提示地址为`0x0000000100000f90`


```
# 退出gdb
(gdb) quit
```

### gdb错误解决方案

可能产生一下错误

```
Unable to find Mach task port for process-id 92788: (os/kern) failure (0x5).
 (please check gdb is codesigned - see taskgated(8))
```

> 最简单的解决方案是重启Mac







## 参考链接

- [段错误(Segmentation fault)产生的原因以及调试方法](http://www.wtango.com/%E6%AE%B5%E9%94%99%E8%AF%AFsegmentation-fault%E4%BA%A7%E7%94%9F%E7%9A%84%E5%8E%9F%E5%9B%A0%E4%BB%A5%E5%8F%8A%E8%B0%83%E8%AF%95%E6%96%B9%E6%B3%95/)
- [Linux环境下段错误的产生原因及调试方法小结](http://www.cnblogs.com/panfeng412/archive/2011/11/06/segmentation-fault-in-linux.html)

