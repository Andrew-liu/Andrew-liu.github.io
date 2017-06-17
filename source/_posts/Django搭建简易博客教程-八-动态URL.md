title: Django搭建简易博客教程(八)-动态URL
date: 2014-12-28 16:40:07
tags: [Python, Django]
---

#动态URL

> 由于Hexo对html代码的解析问题, 暂时先不放代码了, 


运行已经做好的博客框架, 会发现一个问题, 只有一个主页的空盒子, 而大部分时候我们希望能够让每篇博客文章都有一个独立的页面. 

我第一个想到的方法是给每篇博客文章加一个view函数逻辑, 然后设置一个独立的url(我不知道语言比如PHP, 或者web框架rail等是如果解决的, 我是第一次仔细的学习web框架, 也没有前端开发经验), 但是这种方法耦合性太强, 而且用户不友好, 缺点非常多

> Django给我们提供了一个方便的解决方法, 就是`动态URL`

<!--more-->

现在修改my_blog/article/views.py代码:

```
# -*- coding: utf-8 -*-
from django.shortcuts import render
from django.http import HttpResponse
from article.models import Article
from datetime import datetime
from django.http import Http404

# Create your views here.
def home(request):
    post_list = Article.objects.all()  #获取全部的Article对象
    return render(request, 'home.html', {'post_list' : post_list})

def detail(request, id):
    try:
        post = Article.objects.get(id=str(id))
    except Article.DoesNotExist:
        raise Http404
    return render(request, 'post.html', {'post' : post})
```

因为id是每个博文的唯一标识, 所以这里使用id对数据库中的博文进行查找

在my_blog/my_blog/urls.py中修改url设置:

```
from django.conf.urls import patterns, include, url
from django.contrib import admin

urlpatterns = patterns('',
    # Examples:
    # url(r'^$', 'my_blog.views.home', name='home'),
    # url(r'^blog/', include('blog.urls')),

    url(r'^admin/', include(admin.site.urls)),
    url(r'^$', 'article.views.home', name = 'home'),
    url(r'^(?P<id>\d+)/$', 'article.views.detail', name='detail'),
)
```

然后在templates下建立一个用于显示单页博文的界面:



可以发现只需要对home.html进行简单的修改, 去掉循环就可以了.



其中主要改动
- 添加了几个导航按钮, 方便以后添加功能(暂时不添加登陆功能)
- 添加read more按钮

> 比如: 点击到的博客文章标题的对象对应的`id=2`, 这个id被传送到`name=detail`的url中, `'^(?P<id>\d+)/$'`正则表达式匹配后取出id, 然后将id传送到`article.views.detail`作为函数参数, 然后通过get方法获取对应的数据库对象, 然后对对应的模板进行渲染, 发送到浏览器中..

此时重新运行服务器, 然后在浏览器中输入[http://127.0.0.1:8000/](http://127.0.0.1:8000/)点击对应的博客文章题目, 可以成功的跳转到一个独立的页面中


![博客](http://picturebag.qiniudn.com/Snip20141227_6.png)
