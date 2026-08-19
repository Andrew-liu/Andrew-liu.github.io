title: MongoDB入门指南-NoSQL
date: 2015-06-14 19:16:40
tags: [Database, SQL]
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

#简介


NoSQL主要用于Web网站建设, 属于非关系型数据库, 主要解决:

- 对数据库的高并发要求
- 对海量数据的高效率存储和访问
- 对数据库的高可扩展性和高可用性的需求


> 作为一个面向文档的数据库, `MongoDB`是一个更加通用的NoSQL方案. 可以认为它是关系数据库的一个替代方案, 提供了`高性能, 高可用性, 自动扩展`

<!--more-->

- 一个记录在MongoDB被称为一个`文档`
- 一个`文档`由域(field)和值(value)对组成(可以理解为key-value对)
- 值可能又包含其他文档(嵌套概念)
- MongoDB的文档类似与JSON对象
- MongoDB想所有文档存储在`Collections`中(类似与关系数据库中的表概念)
- MongoDB支持`二级索引`

> MongoDB中的文档可以想象成Python中字典数据结构
> 注意掌握索引的应用, 加快大数据时查找速度


#安装

```c
#更新brew和brew cask
$ brew update && brew upgrade brew-cask && brew cleanup && brew cask cleanup
#使用brew安装mongodb
$ brew install mongodb
#在根目录下创建用于数据库写存储数据
$ mkdir -p /data/db
#注意查看该目录的权限, 一定要保证是当前用户权限, 而不是root权限
#运行Mongndb
$ mongod
#使用shell操作
$ mongo
```

#基本操作


MongoDB不需要创建表操作

unicorns以及system.indexes。`system.indexes`在每个数据库中都会创建，它包含了数据库中的索引信息。

**db相关**

```c
# 查询mongoDB中所有的数据库
> show dbs
# 查看当前正在使用的数据库
> db
# 切换数据库, 切换到learn数据库
> use learn
# 删除当前数据库
> db.dropDatabase()
# 查看当前使用的数据库
> db.getName();
# 显示当前db状态
> db.stats();
# 当前db版本
> db.version();
# 查看当前db的链接机器地址
> db.getMongo();
```

**Collection相关**

```c
# 查看当前数据库下的所有collections
> show collections 
# 创建一个聚集集合（table）
> db.createCollection(“collName”, {size: 20, capped: 5, max: 100});
# 得到指定名称的聚集集合（table）
> db.getCollection("account");
# 得到当前db的所有聚集集合
> db.getCollectionNames();
# 显示当前db所有聚集索引的状态
> db.printCollectionStats();
```

##插入

- 使用`db.collection.insert()`向collection(相当于表名)中插入文档(document)
- 返回一个WriteResult对象, 包含操作创建

```c
db.unicorns.insert({name: 'Horny', dob: new Date(1992,2,13,7,47), loves: ['carrot','papaya'], weight: 600, gender: 'm', vampires: 63});
db.unicorns.insert({name: 'Aurora', dob: new Date(1991, 0, 24, 13, 0), loves: ['carrot', 'grape'], weight: 450, gender: 'f', vampires: 43});
db.unicorns.insert({name: 'Unicrom', dob: new Date(1973, 1, 9, 22, 10), loves: ['energon', 'redbull'], weight: 984, gender: 'm', vampires: 182});
db.unicorns.insert({name: 'Roooooodles', dob: new Date(1979, 7, 18, 18, 44), loves: ['apple'], weight: 575, gender: 'm', vampires: 99});
db.unicorns.insert({name: 'Solnara', dob: new Date(1985, 6, 4, 2, 1), loves:['apple', 'carrot', 'chocolate'], weight:550, gender:'f', vampires:80});
db.unicorns.insert({name:'Ayna', dob: new Date(1998, 2, 7, 8, 30), loves: ['strawberry', 'lemon'], weight: 733, gender: 'f', vampires: 40});
db.unicorns.insert({name:'Kenny', dob: new Date(1997, 6, 1, 10, 42), loves: ['grape', 'lemon'], weight: 690,  gender: 'm', vampires: 39});
db.unicorns.insert({name: 'Raleigh', dob: new Date(2005, 4, 3, 0, 57), loves: ['apple', 'sugar'], weight: 421, gender: 'm', vampires: 2});
db.unicorns.insert({name: 'Leia', dob: new Date(2001, 9, 8, 14, 53), loves: ['apple', 'watermelon'], weight: 601, gender: 'f', vampires: 33});
db.unicorns.insert({name: 'Pilot', dob: new Date(1997, 2, 1, 5, 3), loves: ['apple', 'watermelon'], weight: 650, gender: 'm', vampires: 54});
db.unicorns.insert({name: 'Nimue', dob: new Date(1999, 11, 20, 16, 15), loves: ['grape', 'carrot'], weight: 540, gender: 'f'});
db.unicorns.insert({name: 'Dunx', dob: new Date(1976, 6, 18, 18, 18), loves: ['grape', 'watermelon'], weight: 704, gender: 'm', vampires: 165});
```

可以创建一个变量, 保存多个要插入的文档

```c
var mydocuments = [
{name: 'Leia', dob: new Date(2001, 9, 8, 14, 53), loves: ['apple', 'watermelon'], weight: 601, gender: 'f', vampires: 33},
{name: 'Dunx', dob: new Date(1976, 6, 18, 18, 18), loves: ['grape', 'watermelon'], weight: 704, gender: 'm', vampires: 165}
]
#插入到collection
db.unicorns.insert(mydocuments)
```

##查找


查找规则

- 默认情况下, 会返回_id域, 可以使用`_id:0`不映射_id域
- 在`mongo shell`中find默认返回前20个查找到的文档
- 使用`find()`不加任何参数, 等价于find({}), 表示`select * from collection`
- find({<field>:<value>})表示使用一个key-value作为选择条件
- and条件, find({<field>:<value>, <field> : <value>, ...})
- or条件, find( $or : [{<field> : <value>}, {<field> : <value>}, ...])

```c
db.users.find(               #user是collection(相当于关系数据库中的from)
    { age : { $gt : 18} },   #查询条件(相当与where)
    { name : 1, address : 1 }#映射域(相当于select后指明的列)
).limit(5)                   #cursor修改符
```


```c
#查找所有体重超过700磅的雄性独角兽的命令
db.unicorns.find({gender: 'm', weight: {$gt: 700}})
//或者 (效果并不完全一样，仅用来为了演示不同的方法)
db.unicorns.find({gender: {$ne: 'f'}, weight: {$gte: 701}})
```

> `$lt、$lte、$gt、$gte以及$ne`分别表示小于、小于或等于、大于、大于或等于以及不等于,`$exists`操作符用于匹配一个域是否存在, $or操作符并作用于需要进行或操作的数组, 更多查询符可以查看[Query and Projection Operators](http://docs.mongodb.org/manual/reference/operator/query/#query-selectors)


```c
db.unicorns.find({vampires: {$exists: false}})

# 返回所有或者喜欢苹果，或者喜欢橙子，或者体重小于500磅的雌性独角兽
db.unicorns.find({gender: 'f', $or: [{loves: 'apple'}, {loves: 'orange'}, {weight: {$lt: 500}}]})
```

- find的第二个可选参数(`该参数是一个列表`), 用户指明find读取的域

```
#只返回name域(_id域默认总是返回)
db.unicorns.find(null, {name : 1})
#不返回_id域
db.unicorns.find(null, {name : 1, _id : 0})
```

###排序

`1表示升序, -1表示降序`

```
#只查看名字和体重, 并按照体重降序排序
db.unicorns.find(null, {name : 1, weight : 1, _id : 0}).sort({weight : -1})
#优先按照名字升序排序再按照吸血降序排序
db.unicorns.find().sort({name : 1, vampires : -1})
```

###计数

- count命令

```
#返回吸血大于50的个数
db.unicorns.count({vampires : {$gt : 50}})
```

##更新

- `update`根据所给条件找到文档, 然后用`新的文档覆盖找到的整个文档`

**接收参数:**

1. 查找需要的更新文档的匹配条件
2. 更新操作
3. 可选的参数

> 默认情况下, update()更新单个文档, 如果需要更多多个文档, 使用`multi`可选参数


```
#找到原来name所在域, 替换为体重文档
db.unicorns.update({name : 'Roooooodles'}, {weight : 590})
```

- mongo的`$set`修改符可以实现仅改动某个文档的某个值或几个域

```
db.unicorns.update({name: 'Roooooodles'}, {$set : {weight : 590}})
```

- `$inc`修改符激昂一个域值增加一个正或负的值

```
db.unicorns.update({name: 'Pilot'}, {$inc: {vampires: -2}})
```

- `$push`修改符增加一个新的值

```
db.unicorns.update({name: 'Aurora'}, {$push : {love : 'sugar'}})
```

###插新

`upsert`意思为如果存在则更新, 否则插入, 使用时将update第三个参数设置为true


```
#不存在, 插入
db.unicorns.update({name : 'liu'}, {$inc : {hit : 1}}, true)
#再次执行, 则进行+1操作
db.unicorns.update({name : 'liu'}, {$inc : {hit : 1}}, true)
#或者设置
db.unicorns.update({name : 'liu'}, {$inc : {hit : 1}}, {upsert : true})
```

###多重更新

- 将`update`第四个参数设置为true, 表示对所有文档进行修改

```
db.unicorns.update({}, {$set : {vaccinated : true}}, false, true)
#或者设置multi
db.unicorns.update({}, {$set : {vaccinated : true}}, {multi : true})
```

##删除

删除collection中所有文档, 更高效的删除所有文档是使用`drop()`

```
db.unicorns.remove({})
#更高效删除drop(), 不需要任何参数
db.unicorns.drop()
```

删除符合匹配条件的文档

```
db.unicorns.remove({vaccinated : true})
```

#索引

索引的常见组织方式:
- 信息(对象/实例)关联一些特征(`正向索引`)
- 每个特征后关联一批信息(对象/实例)(`反向索引又称倒排索引`)

MongoDB支持高效的索引查询, 索引是一个`数据结构`, 它收集collection中文档特定字段的值, MongoDB的查询优化器能够使用这种数据结构快速的对collection的文档进行查询和排序, 索引通过`B-Tree`实现


> 在MongoDB 3.0.0版本, ensureIndex()创建索引已经过时(不推荐使用), 请使用`createIndex()创建索引`

**索引类型**

- `_id`索引(唯一, 默认创建)
- 用户自定义升序/降序的单field索引
- 复合索引(多field), 其中`field出现顺序与索引排序相关`
- 文本索引, 支持查找collection中的string内容
- 哈希索引,对field对应的值进行哈希作为索引,(only support equality matches)

**索引特点**

- MongoDB中索引是大小写敏感的
- 索引保存在system.indexs中, 运行`db.system.indexs.find()`查看

[MongoDB官方文档的详细描述](http://docs.mongodb.org/manual/core/index-types/)

##创建索引

**基本索引**

```
#当索引不存在时以name filed创建单field索引
db.unicorns.createIndex({name : 1})
```

> field对应的值表示索引类型, 1表示索引为升序, -1表示索引为降序

**唯一索引**

> 如果对要创建唯一索引的field, 存在相同的值(value不唯一), 则不能创建唯一索引

```
#以name field创建唯一索引
db.unicorns.createIndex({name : 1}, {unique : true})
#以a和b field创建唯一复合索引
db.unicorns.createIndex({a : 1, b : 1}, {unique : true})
#如果一定要创建对有相同值的键创建唯一索引, 让系统保存第一条记录, 其余删除, 使用dropDups参数
db.unicorns.createIndex({name : 1}, {unique : true, dropDups : true})
```

**内嵌文档中的key**

MongoDB一个文档中的某个key创建索引

```
db.unicorns.createIndex({"address.city" : 1})
```

**文档索引**

```
db.unicorns.createIndex({"address" :1 })
```

**复合索引**

```
#以address中privince, city, postcode, room作为索引
db.unicorns.createIndex({"address.province" : 1, "address.city" : 1, "address.postcode" : 1, "addresss.room" : 1})
```

**background索引**

默认情况MongoDB在foreground(前景)中创建索引, 防止创建索引时的读写操作(同步阻塞的概念), 背景(background)允许创建索引的同时读写操作, `适用于大批量数据建立索引`

```
#背景创建索引background参数为true
db.unicorns.createIndex({name : 1}, {background : true})
```

**文本索引(important)**

可以对field, 值为字符串或字符串数据的field创建文本索引

```
#对name和subject创建文本索引
db.unicorns.createIndex({name : "text", subject : "text"})
#设置所有包含字符串的field为索引, 并命名为TextIndex
db.unicorns.createIndex(
    {"$**": "text"},
    {name: "TextIndex"}
    )
```

##查看索引

```
#查看当前collection中所有索引
db.unicorns.getIndexs()
```

#删除索引

```
#删除collection中某个索引
db.unicorns.dropIndex({name : 1})
```

#适用场景和不足

**适用场景**

1. 结构不固定, 有数据的嵌套
2. 要求高并发性
3. 频繁的数据水平拆分
4. 内存大于数据量

**不足之处**

1. 比较占用硬盘空间, 性能受内存影响
2. 性能依赖内存, 同时无法指定内存大小, 容易被其他程序占用
3. MongoDB不支持事务, 不支持join
4. 每个Document限制是最大不超过4MB

#pymongo使用


安装pymongo

```
$ pip install pymongo
```

简单使用

```
>>> from pymongo import MongoClient
#使用默认的host和port链接数据库
>>> client = MongoClient()
# 使用获取learn数据库
>>> db = client.learn
# 获取unicorns collections
>>> collection = db.unicorns
# 返回一个查询结果
>>> collection.find_one()
```


#参考链接

- [NoSQL](http://zh.wikipedia.org/zh-cn/NoSQL)
- [MongoDB](http://zh.wikipedia.org/zh-cn/MongoDB)  Key/value硬盘存储
- [Redis](http://zh.wikipedia.org/wiki/Redis)   Key/value RAM存储
- [MemcacheDB](http://en.wikipedia.org/wiki/MemcacheDB)Key/value硬盘存储
- [The Little MongoDB Book](https://github.com/justinyhuang/the-little-mongodb-book-cn/blob/master/mongodb.md)
- [Mongo offical Doc](http://docs.mongodb.org/manual/core/introduction/)
- [SQL to MongoDB Mapping Chart](http://docs.mongodb.org/manual/reference/sql-comparison/)
- [mongoDB 入门指南、示例](http://www.cnblogs.com/hoojo/archive/2011/06/01/2066426.html)
