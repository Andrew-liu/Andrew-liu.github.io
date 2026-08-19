title: Session和Cookie
date: 2015-10-18 22:34:20
tags: [Python, HTML] 
---


本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

~~吐槽start~~

> 上两周都在忙中期考核的事情, 学校发的通知感觉很重视, 所以拿出来了100%的热情对待这件事, 然而最终答辩老师敷衍了事, 各种事不关已无所谓的心态让人反胃. 答辩结果也不进如人意, 不管怎么说, 自己背的锅, 哭着也要走完.

`尽全力去做一件事情, 但不要对结果抱有太高期望, 然后这两件事情总是很难一起做到`

~~吐槽end~~

<!--more-->

## 简介

Session和Cookie两个属于经常会出现在爬虫抓取(更多是Cookie)和Web开发过程中, 一直以来, 我都没搞清楚.

首先考虑一个登录页面到底发生了什么事情:

1. 在浏览器输入登录页面对应的URL, 使用GET方法请求服务器的页面. 然后浏览器渲染出了从服务器端获得的页面资源, 如`login.html`
2. 用户在登录页面输入用户名和密码后(包含在HTML的form标签中), 点击登录按钮, 此时会发出POST请求给远端服务器, 并在服务器调用form标签中制定的action方法进行用户身份验证. 3. 如果验证通过, 页面会自动跳转到其他页面. 如果验证失败会在当前登录页面显示错误信息.

> 然后, 我们常常发现, 当我们关闭当前已登录的页面, 一段时间后, 我们再次访问同样的页面, `会神奇的发现: WTF, 这次已经是登录状态了`

众所周知, HTTP协议是无状态的, 那是什么让页面记住了当前页面的登录状态呢? 答案就是cookie和session. 

> 首先要认识到, Cookie和Session是针对HTTP无状态而产生的两种不同的解决方案. Cookie使客户端(浏览器)保存一些当前网站的身份信息(登陆信息). Session是在服务器端保存一些身份信息


## Cookie

**认识到Cookie存放在客户端非常重要**
怎么来查看Cookie: Chrome设置->显示高级设置->隐私设置-内容设置->所有的Cookie和网站数据

`再来重新看一下登陆过程`

- 用户使用GET方法请求登陆页面的URL, 当服务器返回登陆页面到浏览器时会设置HTTP Header的Set-Cookie字段
- 浏览器存储Set-Cookie字段, 并在之后每次请求服务器时, 都捎带Cookie字段到服务器. 
- Cookie机制是在客户端保持状态的方案
- 客户端接收到多个Cookie时, 同样可以以多个Cookie形式发送
- Cookie有生命周期, 分为回话Cookie和持久Cookie

```
# 使用Httpie工具请求http://www.v2ex.com/时, 得到的响应HTTP报文Header增加了Set-Cookie字段
$ http -h http://www.v2ex.com/
HTTP/1.1 200 OK
Connection: keep-alive
Content-Encoding: gzip
Content-Type: text/html; charset=UTF-8
Date: Sat, 17 Oct 2015 14:56:33 GMT
Set-Cookie: PB3_SESSION="2|1:0|10:1445093721|11:PB3_SESSION|40:djJleDoxMTUuMjMxLjE1Mi4xMzc6Mjc1MTQ1NDI=|b2cdb9e6e42c0ee7e4a9a65124022766efe4a3fb3f1d82565eec6f8e3739eeac"; expires=Thu, 22 Oct 2015 14:55:21 GMT; Path=/
Set-Cookie: V2EX_LANG=zhcn; Path=/
```

Set-Cookie的属性

|属性|  说明|
|---|---|
|NAME=VALUE |赋予Cookie的名称和对应的值, 可以包含多个|
|expires=DATE|  Cookie 的有效期|
|path=PATH| 指定与Cookie关系的Web页面, 可以是目录或者路径（若不指定则默认为文档所在的目录）|
|domain=域名    |指定关联的Web服务器域名（若不指定则默认为创建Cookie的服务器的域名）|
|Secure |仅在HTTPS安全通信时才会发送Cookie|



## Session

Session技术在一定程度上弥补了Cookie存放在本地的不安全性.

- **Session机制是在服务器端保持状态**
- 当一个新用户请求页面时, 服务器会返回Cookie(并创建一个新的Session并生成Session id), 并在Cookie中存放Seesion id对应的值.
- Session的运行依赖Session id, 而Session id是存放在Cookie中的, 通过客户端HTTP中捎带的Session id, 来查找服务器端对应的Session, 实现跟踪会话和验证用户. (如果浏览器禁用了Cookie, 则在URL中附加session id)


Session机制的登录过程:

1. 客户端POST发送登录账户和密码(账户和密码在HTTP数据实体中)
2. 服务器创建新的Session id来识别用户, 验证用户的身份信息后, 将用户的认证状态和Session id保存在服务器端(内存, 硬盘, 数据库都有可能), 在响应HTTP报文中设置Set-Cookie, 并在其中写入Session id
3. 服务器端收到响应报文后, 保存Cookie, 并在下次发送请求时, 捎带包含Session id的Cookie
4. 服务器再次收到Session id可以进行查询对应的用户验证状态.

## 参考链接

- [cookie 和session 的区别详解](http://www.cnblogs.com/shiyangxt/articles/1305506.html)
- [知乎-COOKIE和SESSION有什么区别？](http://www.zhihu.com/question/19786827)

