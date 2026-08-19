title: Python中logging模块
date: 2015-01-20 22:04:04
tags: Python
---

> logging模块为应用程序提供了灵活的手段来记录事件, 错误, 警告, 调试信息. 对这些信息可以收集, 筛选, 写入文件, 发给系统日志等操作.

##1.1. 日志记录级别

每条记录都关联一个级别, 每个级别都有用于发出日志消息的方法, 日志分为五级,

|Level| When it's used| Method|
|---|---|---|
|DEBUG|详细信息, 通常出现在诊断问题|logging.debug()|
|INFO|确认一切按预期运行|logging.info()|
|WARNING(默认等级)|意外发生, 或者说明将来可能发生的问题, 软件可以照常运行|logging.warning()|
|ERROR|更严重的问题, 软件一些功能不能照常运行|logging.error()|
|CRIRICAL|严重错误, 程序本身无法执行|logging.critical()|

<!--more-->


```py
# -*- coding: utf-8 -*-
#!/usr/bin/env python

import logging

logging.critical("软件已废")
logging.error("某些功能不能运行")
logging.warning("有警告")
logging.info("正常")
logging.debug("调试")

###output###
CRITICAL:root:软件已废
ERROR:root:某些功能不能运行
WARNING:root:有警告
```

因为默认等级为WARNING, 所以低于WARNING等级不会输出.

##1.2. 记录日志到文件

对根记录器对象作基本配置的函数为`basicConfig`, 关键字参数:

- filename : 日志消息附加到指定文件中
- filemode : 文件打开模式, 默认为a(追加)
- format : 用于生成日志消息的格式字符串
- datemat : 输出日期和时间的格式字符串
- level : 设置根目录器的级别, 大于等于默认级别的会被处理
- stream : 设置特定的流用于初始化StreamHandler

```py
# -*- coding: utf-8 -*-
#!/usr/bin/env python

import logging, os


logging.basicConfig(filename=os.path.join(os.getcwd(), 'log.txt'), \
    level=logging.INFO, filemode='a', format='%(asctime)s - %(levelname)s: %(message)s')
logging.critical("软件已废")
logging.error("某些功能不能运行")
logging.warning("有警告")
logging.info("正常")
logging.debug("调试")

###output###
2015-01-19 15:56:32,205 - CRITICAL: 软件已废
2015-01-19 15:56:32,205 - ERROR: 某些功能不能运行
2015-01-19 15:56:32,205 - WARNING: 有警告
2015-01-19 15:56:32,205 - INFO: 正常
```

##1.3. 创建Logger对象

getLogger([logname])创建Logger对象
日志记录工作由Logger对象完成, 调用getLogger方法时需要提供Logger名(多次使用相同名称调用getLogger返回同一个对象的引用), Logger之间有阶级层次关系(比如root和root.a, root.a是root的子Logger)


```py
# -*- coding: utf-8 -*-
#!/usr/bin/env python

import logging, os
log1 = logging.getLogger("Liu")
log2 = logging.getLogger("Liu")
print log1, log2
print logging.getLogger(__name__)
print __name__
l

###output###
<logging.Logger object at 0x10606b990> <logging.Logger object at 0x10606b990>  #两个内存地址是相同的
<logging.Logger object at 0x10606b9d0>
__main__
```


##1.4. Logger属性和方法

Logger.propagate  用于指示消息是否传播给父记录器, 默认为True


> - Logger.setLevel(lvl) 设置日志级别, 低于该级别的被忽略
> - Logger.isEnabledFor(lvl)  如果将处理的级别为level的日志消息, 返回True
> - Logger.getEffectiveLevel()  返回Logger的有效级别
> - Logger.getChild(suffix) 返回Logger的后代Logger
> - Logger.addFilter(filt)  给Logger增加筛选器对象
> - Logger.removeFilter(filt) 移除Logger中的筛选器
> - Logger.addHandler(hdlr) 给Logger添加Handler对象
> - Logger.removeHandler(hdlr) 移除Logger中Handler对象
> - Logger.filter(record)
> - Logger.findCaller()
> - Logger.handle(record)
> - Logger.makeRecord(name, lvl, fn, lno, msg, args, exc_info, func=None, extra=None)

**发出消息日志:**
其中msg是信息格式, args和kwargs分别为格式参数

- Logger.debug(msg, *args, **kwargs)
- Logger.info(msg, *args, **kwargs)
- Logger.warning(msg, *args, **kwargs)
- Logger.error(msg, *args, **kwargs)
- Logger.critical(msg, *args, **kwargs)
- Logger.exception(msg, *args, **kwargs) 以ERROR级别记录日志消息
- Logger.log(lvl, msg, *args, **kwargs) lvl参数用户设置日志信息级别

```py
# -*- coding: utf-8 -*-
#!/usr/bin/env python

import logging, os

#基础的跟记录器设置
logging.basicConfig(filename=os.path.join(os.getcwd(), 'log.txt'), \
    level=logging.INFO, filemode='a', format='%(asctime)s - %(levelname)s: %(message)s')
log = logging.getLogger("Liu")  #创建Logger对象
print log.propagate
log.setLevel(logging.INFO)  #设置日志记录级别
log.critical("软件已废")
log.error("某些功能不能运行")
log.warning("有警告")
log.info("正常")
log.debug("调试") #忽略
log.log(logging.CRITICAL, "用户在函数中设置了级别")
log.exception("我是个error")

###File Text###
2015-01-19 16:49:04,492 - CRITICAL: 软件已废
2015-01-19 16:49:04,493 - ERROR: 某些功能不能运行
2015-01-19 16:49:04,493 - WARNING: 有警告
2015-01-19 16:49:04,493 - INFO: 正常
2015-01-19 16:49:04,493 - CRITICAL: 用户在函数中设置了级别
2015-01-19 16:49:04,493 - ERROR: 我是个error
```

##1.5. Handler对象
通过Handler可日志记录(log record)发送到合适的目的的(文件, 控制台, 邮箱等). 通过addHandler方法, Logger对象可以增加0个或者多个Handler对象, [useful-handlers](https://docs.python.org/2/howto/logging.html#useful-handlers)

Handler常用的方法:

- setLevel()  设置日志级别, 低于该级别的被忽略
- setFormatter() Handler选择一个Formatter对象使用
- setFilter() 给Handler设置Filter对象
- removeFilter() 给Handler移除Filter对象

> Handler类主要作为基类被集成,


##1.6. Formatter对象

Formatter对象用来设置最终日志就的顺序结构和内容

```py
#一个参数为信息格式, 第二个参数为日期字符串格式
logging.Formatter.__init__(fmt=None, datefmt=None)
```

> 编写一个同时向控制台和文件中输出日志的日志系统


```py
# -*- coding: utf-8 -*-
#!/usr/bin/env python

import logging

logger = logging.getLogger('Andrew_Liu')  #创建一个Logger对象
logger.setLevel(logging.ERROR)  #设置日志记录级别

fh = logging.FileHandler('log.txt')  #创建一个文件
fh.setLevel(logging.INFO)  #设置日志记录级别

sh = logging.StreamHandler()
sh.setLevel(logging.WARNING)

fmt = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
fh.setFormatter(fmt)
sh.setFormatter(fmt)

logger.addHandler(fh)
logger.addHandler(sh)


logger.critical("软件已废")
logger.error("某些功能不能运行")
logger.warning("有警告")
logger.info("正常")
logger.debug("调试") #忽略


#控制台输出
2015-01-20 21:43:02,037 - Andrew_Liu - CRITICAL - 软件已废
2015-01-20 21:43:02,037 - Andrew_Liu - ERROR - 某些功能不能运行

#文件输出
2015-01-20 21:43:46,098 - Andrew_Liu - CRITICAL - 软件已废
2015-01-20 21:43:46,098 - Andrew_Liu - ERROR - 某些功能不能运行
```

从这个例子中可以明显看到, 在Logger对象和在Handler对象中setLevel的不同之处, 首先日志信息先通过Logger对象的中Level验证, 低于Level忽略, 然后在分别进入不同的Handler对象中的级别进行二次验证.



##1.7.日志记录的配置

基本步骤: 

1. 使用`getLogger()`函数创建各种Logger对象, 正确设置级别等参数
2. 通过实例化各类Handler(FileHandler, StreamHandler, SocketHandler等)创建Handler, 并设置正确的级别
3. 创建消息Formatter对象, 并使用setFormatter方法吧他们附加给Handler对象
4. 使用addHandler()方法将Handler附加给Logger对象



##1.8. 参考链接

- [Logging HOWTO](https://docs.python.org/2/howto/logging.html#handlers)
- [logging官方文档](https://docs.python.org/2/library/logging.html)
