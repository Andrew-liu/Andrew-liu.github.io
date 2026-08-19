title: 理解RESTful API
date: 2016-01-03 18:07:20
tags: Blog
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


## 简介


随着Web开发的不断开发, 低耦合的需求被提上了日程, 催生出了前后端分离的方案. 前后端分离使前端和服务器端相互独立, 细化前后端开发, 于是近几年API架构风行.

> RESTful架构由[Roy Fielding](https://en.wikipedia.org/wiki/Roy_Fielding)在一篇博士论文中提出. `REST, 即Representational State Transfer的缩写, 中文可以译为表现层状态转化


- `表现层`可以理解为资源的表现层, `资源`在网络通信中无处不在, 一段文本, 一张图片, 一个文档都是资源, `JSON是现在最常用的资源表示格式`. (相对于资源, 数据是一种更加抽象的形式)
- `状态转换`可以理解为客户端发起的无状态的HTTP请求, 导致服务器的资源的状态转变, 常用的五种HTTP动词.


<!--more-->

|HTTP动词| 语意|
|---|---|
|GET |从服务器端获取资源(幂等)|
|POST|在服务器新建资源|
|PUT| 在服务器更新资源完整的资源(幂等)|
|PATCH|在服务器更新部分资源|
|DELETE|从服务器删除资源(幂等)|

> `幂等`: 在相同的数据和参数下，执行一次或多次产生的效果（副作用）是一样的


## 设计

- 只提供json作为返回格式

```
Content-Type: application/json; charset=utf-8
```


- `资源`表示一种实体, 所以API设计时应该使用名词, 动词应该方法放在HTTP协议中


```
# GOOD
/api/member/(\d+)/status
GET: 从服务器换取用户状态
PATCH: 在服务器端更新用户状态
# BAD
/api/show/(\d+)/status

# GOOD
/api/message?from=1&to=2
# BAD
/api/message/(\d+)/to/(\d+)

# 获取某用户下的所有gist
GET /users/:username/gists
# 创建一个新的gist
POST /gists
# 删除某个特定的gist
DELETE /gists/:id
# star某个gist
PUT /gists/:id/star
# 搜索, 参数q, sort, order
GET /search/repositories
```


```
# 获取用户资料
GET /api/member/(\d+)/info

# 解禁用户
DELETE /api/member/(\d+)/status
#禁言用户
PATCH /api/member/(\d+)/status

# 获取用户所有私信
GET /api/member/(\d+)/messages

# 获取所有用户的状态
GET /api/members/status
```

- 使用复数表示多个资源

```
# GOOD
/api/members/status

# BAD
/api/account
```

- 过滤信息

```
# filtering 
GET /api/members?client_id=1
# Sorting
GET /api/members?sortby=created&order=desc
# pagination
GET /api/members?page=1&per_page=50
GET /api/github/user/repos?page=2&per_page=100
```

- 将API的版本号放入URL

```
# 挂起状态
/api/member/status

# 只读状态
/api/member/status/v2
```


- Authentication

1. 基本认证机制
2. Token认证机制
3. OAuth


- 增加频率限制, 已认证的帐户已用户为单位, 未认证的账户以ip为单位

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 56
X-RateLimit-Reset: 1372700873
```


## 状态码

| 状态码 |动词| 解释|
|---|---|---|
|200 |*|服务器成功返回用户请求的数据|
|201 |POST/PUT/PATCH|用户新建或修改数据成功|
|202 |*|表示一个请求已经进入后台排队（异步任务）|
|204 |DELETE|用户删除数据成功|
|301|*|永久重定向|
|302|*|临时重定向|
|400 |POST/PUT/PATCH|用户发出的请求有错误，服务器没有进行新建或修改数据的操作|
|401 |*|表示用户没有权限（令牌、用户名、密码错误）|
|403 |*|表示用户得到授权（与401错误相对），但是访问是被禁止的|
|404 |*|用户发出的请求针对的是不存在的记录，服务器没有进行操作|
|406 |GET|用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）|
|410 |GET|用户请求的资源被永久删除，且不会再得到的|
|422 |POST/PUT/PATCH|当创建一个对象时，发生一个验证错误|
|500 |*|服务器发生错误，用户将无法判断发出的请求是否成功|






## 注意事项

1. 更新和创建应该返回一个资源描述, 防止API使用者为了获取更新后的资源而再次调用该API
2. 只返回JSON, 而不是XML
3. 为了防止API滥用, 给API增加某种类型的速率限制(Github对验证通过的用户设置为每小时5000次)



## 参考链接

- [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)
- [RESTful API 设计基本规范](http://ph.in.zhihu.com/w/index_team/web_backend/guides/api-design/)
