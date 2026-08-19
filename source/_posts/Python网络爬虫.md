title: Python爬虫(一)--学习笔记
date: 2014-12-05 18:57:09
tags: Python
---


---


#1. 初探urllib2模块

- 浏览器相当于客户端, 向服务器发出资源请求(node.js充当的是服务器)
- `http`是基于请求和应答机制, 客户端发出请求, 服务器应答请求


```python
import urllib2
response = urllib2.urlopen("http://www.baidu.com/") #使用urlopen进行url请求, 返回应答对象response
html = response.read()  #去读取应答对象的内容
print html

import urllib2
req = urllib2.Request("ftp://www.baidu.com/") #使用url创建一个Request对象
response = urllib2.urlopen(req)
html = response.read()
print html
```

<!--more-->

> urllib2的接口可以处理所有的URL头, 如http, ftp

1. HTTPP请求时, 可以发送data表单数据
2. 设置Headers到http请求(一些站点比喜欢被程序访问, 服务器确认请求的身份通过User-Agent头部, 创建一个请求对象时, 可以给它一个包含头数据的字典)

> HTTPError可以作为页面返回的应答对象response,同样包含read,geturl和info方法

```python
# -*- coding:utf-8 -*-
#发送data表单数据
import urllib2
import urllib

url = "http://www.someserver.com/register.cgi"

values = {
    'name' : 'Andrew',
    'location' : 'NJU',
    'language' : 'Python'
}

#user_agent = 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)' 
#header = {'User-Agent' : user_agent}
data = urllib.urlencode(values) #data数据需要编码成标准形式
print data
req = urllib2.Request(url, data) #发送请求同时传送data表单
#req = urllib2.Request(url, data, headers) #url 表单数据 伪装头部
reponse = urllib2.urlopen(req) #接受反馈数据
html = reponse.read() #读取反馈数据

"""
full_url = url + '?' + data
data = urllib2.open(full_url)
"""


#返回错误码, 200不是错误, 不会引起异常
req = urllib2.Request("http://bbs.csdn.net/callmewhy")
try:
    urllib2.urlopen(req)
except urllib2.HTTPError, e:
    print e.code
    #print e.read()
```

3. urlopen返回的应答对象response有两个常用方法`info()`和`geturl()`

```python
#查看重定向url的真实地址
from urllib2 import Request, urlopen, URLError, HTTPError

old_url = 'http://www.baidu.com'
req = Request(old_url)
response = urlopen(req) &nbsp;
print 'Old url :' + old_url
print 'Real url :' + response.geturl()


#输出:
Old url :http://rrurl.cn/b1UZuP
Real url :http://www.polyu.edu.hk/polyuchallenge/best_of_the_best_elevator_pitch_award/bbca_voting_process.php?voted_team_id=670&section=1&bbca_year_id=3
```

```python
#返回对象的字典, 字典描述了获取的页面情况, 通常是服务器发送的特定头headers
from urllib2 import Request, urlopen, URLError, HTTPError

old_url = 'http://www.baidu.com'
req = Request(old_url)
response = urlopen(req)  
print 'Info():'
print response.info()

#输出
Info():
Date: Fri, 31 Oct 2014 14:45:08 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: Close
Vary: Accept-Encoding
Set-Cookie: BAIDUID=8ACB98CA9413726C3133F5FFA1A24289:FG=1; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
...
```

#2. 异常处理

> HTTPError是URLError的子类

1. `URLError`在没有网络连接或者服务器不存在的情况下产生, 这种情况下异常会带有`reason`属性(不可变的tuple, 包含错误号和错误信息)
2. `HTTPError`, 服务器上每个HTTP应答对象response包含一个`状态码`
HTTP状态码标识HTTP协议所返回的响应状态; `HTTPError`有`code`属性,是服务器发送的相关错误号.

```python
#HTTP状态码
200：请求成功      处理方式：获得响应的内容，进行处理 
201：请求完成，结果是创建了新资源。新创建资源的URI可在响应的实体中得到    处理方式：爬虫中不会遇到 
202：请求被接受，但处理尚未完成    处理方式：阻塞等待 
204：服务器端已经实现了请求，但是没有返回新的信 息。如果客户是用户代理，则无须为此更新自身的文档视图。    处理方式：丢弃
300：该状态码不被HTTP/1.0的应用程序直接使用， 只是作为3XX类型回应的默认解释。存在多个可用的被请求资源。    处理方式：若程序中能够处理，则进行进一步处理，如果程序中不能处理，则丢弃
301：请求到的资源都会分配一个永久的URL，这样就可以在将来通过该URL来访问此资源    处理方式：重定向到分配的URL
302：请求到的资源在一个不同的URL处临时保存     处理方式：重定向到临时的URL 
304 请求的资源未更新     处理方式：丢弃 
400 非法请求     处理方式：丢弃 
401 未授权     处理方式：丢弃 
403 禁止     处理方式：丢弃 
404 没有找到     处理方式：丢弃 
5XX 回应代码以“5”开头的状态码表示服务器端发现自己出现错误，不能继续执行请求    处理方式：丢弃
```


```python
import urllib2

req = urllib2.Request("http://www.zhihu.com/")
try :
    urllib2.urlopen(req)
except urllib2.URLError, e :
    print e.reason

#输出 :
[Errno 8] nodename nor servname provided, or not known
```

##2.1. 处理异常


```python
from urllib2 import Request, urlopen, URLError, HTTPError

req = Request('http://www.zhihu.com')
  
try:  
    response = urlopen(req)  
except URLError, e:  
    if hasattr(e, 'code'):  
        print 'The server couldn\'t fulfill the request.'  
        print 'Error code: ', e.code  
    elif hasattr(e, 'reason'):  
        print 'We failed to reach a server.'  
        print 'Reason: ', e.reason  
else:  
    print 'No exception was raised.'  
    # everything is fine  
```

#3. Openers和Handlers

```python
# -*- coding:utf-8 -*-
import urllib2

#创建一个密码管理者
password_mgr = urllib2.HTTPPasswordMgrWithDefaultRealm()

#添加用户名和密码
top_level_url = "http://example.com/foo/"

#如果知道realm, 可以用来代替None
password_mgr.add_password(None, top_level_url, 'username', 'password')

#创建一个新的Hanlder
handler = urllib2.HTTPBasicAuthHandler(password_mgr)

#创建opener
opener = urllib2.build_opener(handler)

a_url = 'http://www.baidu.com/'

#使用opener获取一个url
opener.open(a_url)

#安装opener, 此后调用urrlib2.urlopen将用自定义opener
urllib2.install_opener(opener)
```

#4. urllib2的使用细节

##4.1. Proxy代理的设置

```python
class urllib2.ProxyHandler([proxies])
Cause requests to go through a proxy. If proxies is given, it must be a dictionary mapping protocol names to URLs of proxies. 

urllib2.install_opener(opener)
Install an OpenerDirector instance as the default global opener. Installing an opener is only necessary if you want urlopen to use that opener

urllib2.build_opener([handler, ...])
Return an OpenerDirector instance, which chains the handlers in the order given. handlers can be either instances of BaseHandler, or subclasses of BaseHandler (in which case it must be possible to call the constructor without any parameters). 
```

urllib2 默认会使用环境变量 http_proxy 来设置 HTTP Proxy, 程序中明确控制 Proxy 而不受环境变量的影响，可以使用代理

```python
import urllib2
enable_proxy = True
proxy_handler = urllib2.ProxyHandler({"http" : 'http://some-proxy.com:8080'})
null_proxy_handler = urllib2.ProxyHandler({})
if enable_proxy:
    opener = urllib2.build_opener(proxy_handler)
else:
    opener = urllib2.build_opener(null_proxy_handler)
urllib2.install_opener(opener)  #urllib2.install_opener() 会设置 urllib2 的全局 opener
```

##4.2. Timeout设置


```python
The optional timeout parameter specifies a timeout in seconds for blocking operations like the connection attempt (if not specified, the global default timeout setting will be used). This actually only works for HTTP, HTTPS and FTP connections

import urllib2
response = urllib2.urlopen('http://www.google.com', timeout=10)
```

##4.3. 加入特定的Header

```
import urllib2
request = urllib2.Request('http://www.baidu.com/')
request.add_header('User-Agent', 'fake-client')
response = urllib2.urlopen(request)
print response.read()


#或者直接使用
request = urllib2.Request('http://www.baidu.com/', {'User-Agent': 'fake-client'})
```

注意特殊的Header

- User-Agent : 有些服务器或 Proxy 会通过该值来判断是否是浏览器发出的请求
- Content-Type : 在使用 REST 接口时，服务器会检查该值，用来确定 HTTP Body 中的内容该怎样解析。常见的取值有：
- application/xml ： 在 XML RPC，如 RESTful/SOAP 调用时使用
- application/json ： 在 JSON RPC 调用时使用
- application/x-www-form-urlencoded ： 浏览器提交 Web 表单时使用
在使用服务器提供的 RESTful 或 SOAP 服务时， Content-Type 设置错误会导致服务器拒绝服务


##4.4. Redirect

只要检查一下 Response 的 URL 和 Request 的 URL 是否一致就可以了。

```python
import urllib2
my_url = 'http://www.baidu.cn'
response = urllib2.urlopen(my_url)
redirected = response.geturl() == my_url
print redirected

my_url = 'http://rrurl.cn/b1UZuP'
response = urllib2.urlopen(my_url)
redirected = response.geturl() == my_url
print redirected
```

如果不想自动 redirect，除了使用更低层次的 httplib 库之外，还可以自定义`HTTPRedirectHandler` 类


##4.5. Cookie

urllib2 对 Cookie 的处理也是自动的。如果需要得到某个 Cookie 项的值, 可以像下面一样

```python
import urllib2
import cookielib
cookie = cookielib.CookieJar()
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))
response = opener.open('http://www.baidu.com')
for item in cookie:
    print 'Name = '+item.name
    print 'Value = '+item.value
    
Name = BAIDUID
Value = 6AD640C70CF0772950084D47494ADF9F:FG=1
Name = BAIDUPSID
Value = 6AD640C70CF0772950084D47494ADF9F
Name = H_PS_PSSID
Value = 5015_10159_1442_9591_7800_10194_10251_9932_10211_10120_10210_10016_10178_9498_10051_10066_9770_10096_10008_10235_10257_9978_9023
Name = BDSVRTM
Value = 0
Name = BD_HOME
Value = 0
```

##4.6. 得到 HTTP 的返回码

对于正常访问的网页,urlopen 返回的 response 对象的 getcode() 方法就可以得到 HTTP 的返回码。但对其它返回码来说，urlopen 会抛出异常。这时候，就要检查异常对象的 code 属性

```python
def get_code():
    try:
        response = urllib2.urlopen("http://www.baidu.com")
    except urllib2.HTTPError, e:
        print e.code
    print response.getcode()
```

##4.7. Debug log
使用 urllib2 时，可以通过下面的方法把 debug Log 打开，这样收发包的内容就会在屏幕上打印出来，方便调试，有时可以省去抓包的工作

```python
def debug_log():
    http_handler = urllib2.HTTPHandler(debuglevel = 1)
    https_handler = urllib2.HTTPSHandler(debuglevel = 1)
    opener = urllib2.build_opener(http_handler, https_handler)
    urllib2.install_opener(opener)
    response = urllib2.urlopen("http://www.baidu.com")

send: 'GET / HTTP/1.1\r\nAccept-Encoding: identity\r\nHost: www.baidu.com\r\nConnection: close\r\nUser-Agent: Python-urllib/2.7\r\n\r\n'
reply: 'HTTP/1.1 200 OK\r\n'
header: Date: Sat, 29 Nov 2014 12:48:32 GMT
header: Content-Type: text/html; charset=utf-8
header: Transfer-Encoding: chunked
header: Connection: Close
header: Vary: Accept-Encoding
header: Set-Cookie: BAIDUID=8CD4DE8C39725B1B6C054E5211DCA422:FG=1; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
header: Set-Cookie: BAIDUPSID=8CD4DE8C39725B1B6C054E5211DCA422; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
header: Set-Cookie: BDSVRTM=0; path=/
header: Set-Cookie: BD_HOME=0; path=/
header: Set-Cookie: H_PS_PSSID=10148_10204_10160_1423_9993_7802_10194_9475_10121_10209_10016_10179_9499_10051_10218_10065_9769_10096_10007_10236_10257_9978_9024; path=/; domain=.baidu.com
header: P3P: CP=" OTI DSP COR IVA OUR IND COM "
header: Cache-Control: private
header: Cxy_all: baidu+d0a19f3803227c13f67815859a8916ed
header: Expires: Sat, 29 Nov 2014 12:48:06 GMT
header: X-Powered-By: HPHP
header: Server: BWS/1.1
header: BDPAGETYPE: 1
header: BDQID: 0xdb113cef0017ad4f
header: BDUSERID: 0
```

##4.8. 表单的处理

用firefox+httpfox插件来看看自己到底发送了些什么包。
先找到自己发的POST请求，以及POST表单项。

```python
# -*- coding: utf-8 -*-
import urllib
import urllib2
postdata=urllib.urlencode({
&nbsp;&nbsp;&nbsp; 'username':'汪小光',
&nbsp;&nbsp;&nbsp; 'password':'why888',
&nbsp;&nbsp;&nbsp; 'continueURI':'http://www.verycd.com/',
&nbsp;&nbsp;&nbsp; 'fk':'',
&nbsp;&nbsp;&nbsp; 'login_submit':'登录'
})
req = urllib2.Request(
&nbsp;&nbsp;&nbsp; url = 'http://secure.verycd.com/signin',
&nbsp;&nbsp;&nbsp; data = postdata
)
result = urllib2.urlopen(req)
print result.read() 
```

##4.9. 伪装成浏览器访问

某些网站反感爬虫的到访，于是对爬虫一律拒绝请求
这时候我们需要伪装成浏览器，这可以通过修改http包中的header来实现

```python
def imitate_browser():
    post_data = urllib.urlencode({
        'username': '汪小光',
        'password': 'why888',
        'continueURI': 'http://www.verycd.com/',
        'fk': '',
        'login_submit': '登陆'
    })
    headers = {
    'User-Agent': 'Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.1.6) Gecko/20091201 Firefox/3.5.6' 
    }
    req = urllib2.Request(
    url = 'http://secure.verycd.com/signin/*/http://www.verycd.com/',
    data = post_data, 
    headers = headers
    )
    result = urllib2.urlopen(req)
    print result.read()
```


##4.10. 对付"反盗链"

某些站点有所谓的反盗链设置，其实说穿了很简单，
就是检查你发送请求的header里面，referer站点是不是他自己，
所以我们只需要像把headers的referer改成该网站即可

```python
def reverse_link():
    """

    反盗链, 查看头部referer是否为访问的网站
    """
    post_data = urllib.urlencode({
        'username': '汪小光',
        'password': 'why888',
        'continueURI': 'http://www.verycd.com/',
        'fk': '',
        'login_submit': '登陆'
    })
    headers = {
    'Referer': 'http://www.cnbeta.com/articles'
    }
    req = urllib2.Request(
    url = 'http://secure.verycd.com/signin/*/http://www.verycd.com/',
    data = post_data, 
    headers = headers
    )
    result = urllib2.urlopen(req)
    print result.read()
```



> 本文为学习[Python爬虫入门教程](http://blog.csdn.net/column/details/why-bug.html)笔记
