title: C++实现简单的反射机制
date: 2015-12-17 20:27:51
tags: C++
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.



# 反射机制

在Python中, 通常RPC中经常看到一种场景, 通过协议解析客户端请求后, 拿到要调用的函数的字符串, 通过字典映射得到对应的可调用的函数, 其中利用了Python解释性语言的特性. 这个特性有一个名字叫`反射`

> 反射是通过一个字符串名来做创建, 调用, 获取信息等操作. 


C++是一种静态语言, 并不支持这种特性, 但是我们可以自己实现这种`反射机制`.

<!--more-->

**反射的常用场景**

- 序列化和数据绑定
- 远程方法调用(RPC)
- 对象关系设置

> 为什么静态方法中, 通过引用的方式返回后的对象不同于原来的对象


## 实现

**实现思路**

1. 通过`Map`数据结构存储类名到类型的映射
2. 通过一个公有的方法来取得映射关系对象
3. 通过字符串key找到对应的类类型并生成相应的类的对象




```


// 工厂的基类
class BaseFactory {
public:
    virtual boost::any Init() const {
        std::cout << "BaseFactory" << std::endl;
        return boost::any();  // 这里返回我们想要在反射后调用类的实例
    }
};

// 工厂的继承类
class TestFactory : public BaseFactory {
    public:
        boost::any Init() const {
            std::cout << "TestFactory" << std::endl;
            return boost::any();
        }
};

// reflect保存映射关系的对象
map<string, BaseFactory*>& GetReflectorMap();

// register and get method
boost::any GetInstanceFromMap(const string& class_name);
void RegisterInstanceIntoMap(const string& class_name, BaseFactory*);

#endif /* _REFLECTOR_H_ */
```

在Map的Key中存储类型对象字符串, 通过Key-value将字符串和类型对应起来, Value中存储工厂基类的指针或者引用. `为什么使用指针呢,` 这样可以使用C++的动态绑定的特性(保证基类指针在调用成员方法时根据真实类型进行成员方法调用)

- `RegisterInstanceIntoMap`类名字符串和对应的继承类类型注册到Map关系映射中
- `GetInstanceFromMap`通过字符串获得并调用对象的类实例的`Init方法`, 可方法由自己定义逻辑.


> 这里给出一种基本的实现思路, 还有更多细节需要完善

- [Github源码链接](https://github.com/Andrew-liu/Reflect)

## 参考链接

- [一个例子让你了解Java反射机制](http://blog.csdn.net/ljphhj/article/details/12858767)

