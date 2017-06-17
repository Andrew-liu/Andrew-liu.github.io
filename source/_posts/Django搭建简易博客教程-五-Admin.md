title: Django搭建简易博客教程(五)-Admin
date: 2014-12-27 09:40:48
tags: [Python, Django]
---

#Admin简介
---

Django有一个优秀的特性, 内置了Django admin后台管理界面, 方便管理者进行添加和删除网站的内容.


#设置Admin
---

> 新建的项目系统已经为我们设置好了后台管理功能

可以在my_blog/my_blog/setting.py中查看

```
INSTALLED_APPS = (
    'django.contrib.admin',  #默认添加后台管理功能
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'article'
)
```

<!--more-->

同时也已经添加了进入后天管理的url, 可以在my_blog/my_blog/urls.py中查看

```
from django.conf.urls import patterns, include, url
from django.contrib import admin

urlpatterns = patterns('',
    # Examples:
    # url(r'^$', 'my_blog.views.home', name='home'),
    # url(r'^blog/', include('blog.urls')),

    url(r'^admin/', include(admin.site.urls)),  #可以使用设置好的url进入网站后台
    url(r'^$', 'article.views.home'),
)
```




#创建超级用户
---

使用如下命令账号创建超级用户(如果使用了`python manage.py syncdb`会要求你创建一个超级用户, `该命令已经过时, 不再推荐使用`)

```
$ python manage.py createsuperuser
Username (leave blank to use 'andrew_liu'): root
Email address:
Password:
Password (again):
Superuser created successfully.
```

输入用户名, 邮箱, 密码就能够创建一个超级用户
现在可以在浏览器中输入[127.0.0.1:8000/admin](http://127.0.0.1:8000/admin)输入账户和密码进入后台管理, 如下:

![后台](http://picturebag.qiniudn.com/Snip20141226_3.png)

![进入](http://picturebag.qiniudn.com/Snip20141226_4.png)

但是你会发现并没有数据库信息的增加和删除, 现在我们在my_blog/article/admin.py中增加代码:

```
from django.contrib import admin
from article.models import Article

# Register your models here.
admin.site.register(Article)
```

保存后, 再次刷新页面, [127.0.0.1:8000/admin](http://127.0.0.1:8000/admin)

![成功](http://picturebag.qiniudn.com/Snip20141226_5.png)

对于管理界面的外观的定制还有展示顺序的修改就不详细叙述了, 感兴趣的可以查看官方文档...


#使用第三方插件

Django现在已经相对成熟, 已经有许多不错的可以使用的第三方插件可以使用, 这些插件各种各样, 现在我们使用一个第三方插件使后台管理界面更加美观, 目前大部分第三方插件可以在[Django Packages ](https://www.djangopackages.com/)中查看,

尝试使用[django-admin-bootstrap](https://github.com/douglasmiranda/django-admin-bootstrap)美化后台管理界面


##安装

```
$ sudo pip install bootstrap-admin
```

##配置

然后在my_blog/my_blog/setting.py中修改`INSTALLED_APPS`

```

INSTALLED_APPS = (
    'bootstrap_admin',  #一定要放在`django.contrib.admin`前面
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'article',
)

from django.conf import global_settings
TEMPLATE_CONTEXT_PROCESSORS = global_settings.TEMPLATE_CONTEXT_PROCESSORS + (
    'django.core.context_processors.request',
)
BOOTSTRAP_ADMIN_SIDEBAR_MENU = True
```


保存后, 再次刷新页面, [127.0.0.1:8000/admin](http://127.0.0.1:8000/admin)

![第三方](http://picturebag.qiniudn.com/Snip20141226_6.png)


> 界面是不是美腻了许多...
