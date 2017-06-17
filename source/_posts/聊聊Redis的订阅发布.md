title: 聊聊Redis的订阅发布
date: 2016-01-30 22:32:11
tags: Database
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


## 为什么做订阅分布?

随着业务复杂, 业务的项目依赖关系增强, 使用消息队列帮助系统`降低耦合度`.

- 订阅分布本身也是一种生产者消费者模式, 订阅者是消费者, 发布者是生产者.
- 订阅发布模式, 发布者发布消息后, 只要有订阅方, 则多个订阅方会收到同样的消息
- 生产者消费者模式,  生产者往队列里放入消息, 由多个消费者对一条消息进行抢占.

- 订阅分布模式可以将一些不着急完成的工作放到其他进程或者线程中进行离线处理.

<!--more-->

## Redis中的订阅发布

> Redis中的订阅发布模式, 当没有订阅者时, 消息会被直接丢弃(Redis不会持久化保存消息)



## Redis生产者消费者

生产者使用Redis中的list数据结构进行实现, 将待处理的消息塞入到消息队列中.


```
class Producer(object):

    def __init__(self, host="localhost", port=6379):
        self._conn = redis.StrictRedis(host=host, port=port)
        self.key = "test_key"
        self.value = "test_value_{id}"

    def produce(self):
        for id in xrange(5):
            msg = self.value.format(id=id)
            self._conn.lpush(self.key, msg)
```

消费者使用`redis中brpop`进行实现, brpop会从list头部消息, 并能够设置超时等待时间.

```
class Consumer(object):

    def __init__(self, host="localhost", port=6379):
        self._conn = redis.StrictRedis(host=host, port=port)
        self.key = "test_key"

    def consume(self, timeout=0):
        # timeout=0 表示会无线阻塞, 直到获得消息
        while True:
            msg = self._conn.brpop(self.key, timeout=timeout)
            process(msg)


def process(msg):
    print msg

if __name__ == '__main__':
    consumer = Consumer()
    consumer.consume()
# 输出结果
('test_key', 'test_value_1')
('test_key', 'test_value_2')
('test_key', 'test_value_3')
('test_key', 'test_value_4')
('test_key', 'test_value_5')

```



## Redis中订阅发布

在Redis Pubsub中, 一个频道(channel)相当于一个消息队列

```
class Publisher(object):

    def __init__(self, host, port):
        self._conn = redis.StrictRedis(host=host, port=port)
        self.channel = "test_channel"
        self.value = "test_value_{id}"

    def pub(self):
        for id in xrange(5):
            msg = self.value.format(id=id)
            self._conn.publish(self.channel, msg)
```

其中`get_message`使用了`select` IO多路复用来检查socket连接是否是否可读.

```
class Subscriber(object):

    def __init__(self, host="localhost", port=6379):
        self._conn = redis.StrictRedis(host=host, port=port)
        self._pubsub = self._conn.pubsub()  # 生成pubsub对象
        self.channel = "test_channel"
        self._pubsub.subscribe(self.channel)

    def sub(self):
        while True:
            msg = self._pubsub.get_message()
            if msg and isinstance(msg.get("data"), basestring):
                process(msg.get("data"))
    
    def close(self):
        self._pubsub.close()

# 输出结果
test_value_1
test_value_2
test_value_3
test_value_4
test_value_5
```


## Java Jedis踩过的坑

在Jedis中订阅方处理是采用同步的方式, 看源码中[PubSub模块的process函数](https://github.com/xetorthio/jedis/blob/master/src/main/java/redis/clients/jedis/JedisPubSub.java)

在`do-while`循环中, 会等到当前消息处理完毕才能够处理下一条消息, 这样会导致当入队列消息量过大的时候, redis链接被强制关闭.

解决方案: 将整个处理函数改为异步的方式.

