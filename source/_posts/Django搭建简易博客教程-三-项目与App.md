title: Django搭建简易博客教程(三)-项目与App
date: 2014-12-22 19:20:15
tags: [Django, Python]
---

现在正式开始吧, 我们创建一个名为`my_blog`的Django项目

**创建项目的指令如下:**

```
$ django-admin.py startproject my_blog
```

现在来看一下整个项目的文件结构

```
$ tree my_blog   #打印树形文件结构

my_blog
├── manage.py
└── my_blog
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py

1 directory, 5 files
```

<!--more-->

#建立Django app
---

>　在`Django`中的app我认为就是一个功能模块, 与其他的web框架可能有很大的区别, 将不能功能放在不同的app中, 方便代码的复用

建立一个`article` app

```
$ python manage.py startapp article
```

现在让我们重新看一下整个项目的结构

```
── article
│   ├── __init__.py
│   ├── admin.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── db.sqlite3
├── manage.py
├── my_blog
    ├── __init__.py
    ├── __pycache__
    │   ├── __init__.cpython-34.pyc
    │   ├── settings.cpython-34.pyc
    │   ├── urls.cpython-34.pyc
    │   └── wsgi.cpython-34.pyc
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```
并在my_blog/my_blog/setting.py下添加新建app

```
INSTALLED_APPS = (
    ...
    'article',  #这里填写的是app的名称
)
```


#运行程序
---

```
$ python manage.py runserver   #启动Django中的开发服务器
```

```
#如果运行上面命令出现以下提示
You have unapplied migrations; your app may not work properly until they are applied.
Run 'python manage.py migrate' to apply them.
#请先使用下面命令
python manage.py migrate
#输出如下信息
Operations to perform:
  Apply all migrations: contenttypes, sessions, admin, auth
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying sessions.0001_initial... OK
```

运行成功后,会显示如下信息

```
System check identified no issues (0 silenced).
December 21, 2014 - 08:56:00
Django version 1.7.1, using settings 'my_blog.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

现在可以启动浏览器, 输入[http://127.0.0.1:8000/](http://127.0.0.1:8000/), 当出现
`It worked!`
`Congratulations on your first Django-powered page` 

说明你成功走出了第一步!
