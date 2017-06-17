title: Python爬虫(二)--豆瓣抓站小计
date: 2014-12-05 18:58:43
tags: Python
---

---

#1. 豆瓣抓站流程

1. 分析url特征(`菜鸟阶段`)
2. 对需要抓取的数据设计正则表达式
3. 处理HTML中一些特征字符,换行符等

> 注意异常的处理和字符编码的处理

#2. 实现的功能

**简单的实现了抓取豆瓣电影Top100的电影名称**


#3. 后期工作展望

- 抓取更多的有用数据(如:准确抓取导演, 抓取一个电影评论)
- 使用多线程爬虫
- 学习第三方的爬虫框架(`Scrapy`)
- 深入理解HTML编码和文本处理

<!--more-->

#4. 输出结果

```
Top1 肖申克的救赎
Top2 这个杀手不太冷
Top3 阿甘正传
Top4 霸王别姬
Top5 美丽人生
...
Top96 菊次郎的夏天
Top97 驯龙高手
Top98 真爱至上
Top99 致命ID
Top100 超脱
```

#5. 豆瓣抓站源代码

```python
#!/usr/bin/python2
# -*- coding:utf-8 -*-
"""
一个简单的Python爬虫, 用于抓取豆瓣电影Top前100的电影的名称

Anthor: Andrew Liu
Version: 0.0.1
Date: 2014-12-04
Language: Python2.7.8
Editor: Sublime Text2
Operate: 具体操作请看README.md介绍
"""
import string
import re
import urllib2

class DouBanSpider(object) :
    """类的简要说明

    本类主要用于抓取豆瓣前100的电影名称
    
    Attributes:
        page: 用于表示当前所处的抓取页面
        cur_url: 用于表示当前争取抓取页面的url
        datas: 存储处理好的抓取到的电影名称
        _top_num: 用于记录当前的top号码
    """

    def __init__(self) :
        self.page = 1
        self.cur_url = "http://movie.douban.com/top250?start={page}&filter=&type="
        self.datas = []
        self._top_num = 1
        print "豆瓣电影爬虫准备就绪, 准备爬取数据..."

    def get_page(self, cur_page) :
        """

        根据当前页码爬取网页HTML

        Args: 
            cur_page: 表示当前所抓取的网站页码

        Returns:
            返回抓取到整个页面的HTML(unicode编码)

        Raises:
            URLError:url引发的异常
        """
        url = self.cur_url
        try :
            my_page = urllib2.urlopen(url.format(page = (cur_page - 1) * 25)).read().decode("utf-8")
        except urllib2.URLError, e :
            if hasattr(e, "code"):
                print "The server couldn't fulfill the request."
                print "Error code: %s" % e.code
            elif hasattr(e, "reason"):
                print "We failed to reach a server. Please check your url and read the Reason"
                print "Reason: %s" % e.reason
        return my_page

    def find_title(self, my_page) :
        """

        通过返回的整个网页HTML, 正则匹配前100的电影名称

        
        Args:
            my_page: 传入页面的HTML文本用于正则匹配
        """
        temp_data = []
        movie_items = re.findall(r'<span.*?class="title">(.*?)</span>', my_page, re.S)
        for index, item in enumerate(movie_items) :
            if item.find("&nbsp") == -1 :
                temp_data.append("Top" + str(self._top_num) + " " + item)
                self._top_num += 1
        self.datas.extend(temp_data)
    
    def start_spider(self) :
        """

        爬虫入口, 并控制爬虫抓取页面的范围
        """
        while self.page <= 4 :
            my_page = self.get_page(self.page)
            self.find_title(my_page)
            self.page += 1

def main() :
    print """
        ###############################
            一个简单的豆瓣电影前100爬虫
            Author: Andrew_liu
            Version: 0.0.1
            Date: 2014-12-04
        ###############################
    """
    my_spider = DouBanSpider()
    my_spider.start_spider()
    for item in my_spider.datas :
        print item
    print "豆瓣爬虫爬取结束..."

if __name__ == '__main__':
    main()
```

#6. 参考链接

[个人使用的Python编码规范](http://andrewliu.tk/2014/11/28/Python%E7%BC%96%E7%A0%81%E8%A7%84%E8%8C%83/)
[python正则表达式小计](http://andrewliu.tk/2014/10/26/2014-10-26-Python%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/)


