title: Redis使用入门
date: 2015-06-27 14:10:27
tags: Database
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

#Redis Introduction


`Redis`是一个用C语言实现的key-value store. 除了最基础的基于字符串的键值对，Redis可以是更加复杂的数据结构，所以redis也常被称为是一个data structure server(数据结构服务器)

Redis支持的数据结构列表:

- String(字符串)
- List(列表, 基本的`链表`)
- Set(元素唯一的, 无序的集合)
- Sorted Set(类似于Set, 但是每个字符串元素关联一个浮点数score, 用于排序)
- Hash(哈希)
- Bitmap
- HyperLogLogs

> 本篇没有涵盖Bitmap和HyperLogLogs的使用

<!--more-->
#Redis Install

在[Redis下载页面](http://redis.io/download)下载需要版本的redis

```
# 解压源文件
$ tar xzf redis-3.0.2.tar.gz
# 进入redis文件夹
$ cd redis-3.0.2
# 编译源文件
$ make
# 测试
$ make test

# 提示以下信息表示安装成功
\o/ All tests passed without errors!
Cleanup: may take some time... OK
```

**运行Redis的服务器端**

```
$ /src/redis-server
```


**运行Redis的交互式客户端**

> 用过python应该都理解交互式的意义吧

```
$ /src/redis-cli
```

**在.zshrc设置快捷启动**

```
$ vim ~/.zshrc
# 在最后一行添加

# Redis shortcur
alias reds="/Users/andrew_liu/Development/redis-3.0.2/src/redis-server"
alias redc="/Users/andrew_liu/Development/redis-3.0.2/src/redis-cli"

# 保存后, 运行(Esc, :wq把保存并退出)
$ source ~/.zshrc

# 启动Redis服务器
$ reds

# 启动Redis交互式客户端
$ redc
```

#Redis keys

Redis keys是二进制安全的, 可以使用任何二进制序列作为key, 空的字符串也是有效地key

#Redis Strings

使用字符串作为value时, 我们在string key和string value之间建立了映射关系

- value不能大小不能操作`512MB`
- `SET`命令设置键值对, 重复设置key的value, 会覆盖原来的value
- `GET`命令通过key检索value
- `INRC`命令将给定key的value看做整型, 实现`加一`操作, 这个命令是`原子性的`(原子性中是事务ACID特性之一)
- `GETSET`命令通过新的value设置key, 并返回旧的value
- `MSET`和`MGET`命令分别用于同时设置或检索多个key
- `EXISTS`命令, 如果给定的key存在返回1, 否则返回0
- `DEL`命令删除key-value对

```
127.0.0.1:6379> set mykey hello
OK
127.0.0.1:6379> get mykey
"hello"
127.0.0.1:6379> get mykey
"1"
127.0.0.1:6379> incr mykey
(integer) 2
127.0.0.1:6379> getset mykey how
"2"
127.0.0.1:6379> get mykey
"how"
127.0.0.1:6379> mset a 10 b 20 c 30
OK
127.0.0.1:6379> mget a b c
1) "10"
2) "20"
3) "30"
127.0.0.1:6379> exists a
(integer) 1
127.0.0.1:6379> del a
(integer) 1
127.0.0.1:6379> exists a
(integer) 0
```

#Redis 定时器

- `EXPIRE`命令提供定时器功能, 设置key只能存在固定时间, 时间到key会被删除(`以秒为单位`)
- `TTL`用于查询key还有多长时间被删除.
TTL返回`-2`表示key已经被删除, TTL返回`-1`表示key不会expire


```
127.0.0.1:6379> set key value
OK
127.0.0.1:6379> get key
"value"
127.0.0.1:6379> expire key 1000
(integer) 1
127.0.0.1:6379> ttl key
(integer) 995
# 1000秒后, 被删除
127.0.0.1:6379> get key
(nil)
```

#Redis Lists

> Redis通过链表来实现List

**链表的实现优点是快速的插入和删除, 缺点是检索相对数组实现List慢**

- `LPUSH`命令从List的头部插入一个或多个value, 返回插入后的list长度
- `RPUSH`命令从List的尾部插入一个或多个value, 返回插入后的list长度
- `LLEN`命令返回List的长度
- `LRANGE`命令从List取出固定一段`子List`(类似于python中的切片操作), 有两个索引参数, 都可以为负数
- `LPOP`命令从尾部删除value
- `RPOP`命令从头部删除value, 没有value可删时返回nil

```
127.0.0.1:6379> rpush list A
(integer) 1
127.0.0.1:6379> rpush list B
(integer) 2
127.0.0.1:6379> lpush list C
(integer) 3
127.0.0.1:6379> llen list
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "C"
2) "A"
3) "B"
127.0.0.1:6379> rpush list a b c
(integer) 6
127.0.0.1:6379> rpop list
"c"
127.0.0.1:6379> lpop list
"C"
```



#Redis Sets

> value为Set, 表示无需的字符串集合

- `SADD`命令向set中增加元素, 如果元素已存在则返回0, 否则返回1
- `SMEMERS`命令返回set集合, 返回的集合与插入时的顺序无关
- `SISMEMBER`命令测试元素是否在set中, 在返回1, 不在返回0
- `SREM`移除set中的元素
- `SUNION`命令链接多个set集合

```
127.0.0.1:6379> sadd mset 1
(integer) 1
127.0.0.1:6379> sadd mset 1
(integer) 0
127.0.0.1:6379> sadd mset 2
(integer) 1
127.0.0.1:6379> smembers mset
1) "1"
2) "2"
127.0.0.1:6379> sismember mset 1
(integer) 1
127.0.0.1:6379> sismember mset 3
(integer) 0
```

#Redis Sorted Sets

> Sorted Set类似于Set和Hash的混合, Sorted Set中的元素也是唯一不重复的字符窜元素, `Set中的元素无需, Sorted Set中通过浮点数score排序`, Sorted Set通过`跳表`和`哈希表`实现, 增加一个元素的复杂度`O(log(N))`

排序原则: 

- score不同时, A.score > B.score => A > B
- score相同时, A.score = B.score => A和B通过比较字符串的`字典序`进行排序


- `ZADD`命令增加带score的元素, 可以传入多个score-value对
- `ZRANGE`命令类似于`LRANGE`操作, 
- `ZREVRANGE`命令反序输出Sorted set元素片段

```
127.0.0.1:6379> zadd sset 1991 "liu"
(integer) 1
127.0.0.1:6379> zadd sset 1991 "andrew"
(integer) 1
127.0.0.1:6379> zadd sset 1992 "dinosaur"
(integer) 1
127.0.0.1:6379> zadd sset 1993 "bingo"
(integer) 1
127.0.0.1:6379> zadd sset 1994 "hello" 1995 "world"
(integer) 2
127.0.0.1:6379> zrange sset 0 -1
1) "andrew"
2) "liu"
3) "dinosaur"
4) "bingo"
5) "hello"
6) "world"
127.0.0.1:6379> zrevrange sset 0 -1
1) "world"
2) "hello"
3) "bingo"
4) "dinosaur"
5) "liu"
6) "andrew"
```

#Redis Hashes

- `HSET`命令设置key中field和field应对的value
- `HMSET`命令用于设置哈希中的多个field
- `HGET`命令用于检索单个field
- `HMGET`命令用于检索多个filed
- `HGETALL`命令获得key的所有的field和field对应的value

```
127.0.0.1:6379> hset user:1001 name "jane"
(integer) 1
127.0.0.1:6379> hset user:1001 email "jane@gamil.com"
(integer) 1
127.0.0.1:6379> hgetall user:1001
1) "name"
2) "jane"
3) "email"
4) "jane@gamil.com"
127.0.0.1:6379> hmset user:1000 username andrewliu birthday 2015 verfield 1
OK
127.0.0.1:6379> hget user:1000 username
"andrewliu"
127.0.0.1:6379> hgetall user:1000
1) "username"
2) "andrewliu"
3) "birthday"
4) "2015"
5) "verfield"
6) "1"
127.0.0.1:6379> hmget user:1000 username birthday
1) "andrewliu"
2) "2015"
```




#Reference

- [Redis 设计与实现](http://redisbook.com/)
- [An introduction to Redis data types and abstractions](http://redis.io/topics/data-types-intro)

