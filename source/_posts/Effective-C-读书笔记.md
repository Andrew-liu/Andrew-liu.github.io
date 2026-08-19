title: Effective C++ 读书笔记
date: 2015-09-11 15:33:42
tags: 读书笔记
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


> 最喜欢书中的一句话: `所谓最佳系统设计, 取决于系统希望做什么事情, 包括现在与未来`.


## Use const whenever possible

```c
void TestConst() {
    char greeting[] = "Hello";  // length = 6
    char test[] = "world";
    char *pt = greeting;  // 指向数据中第一个元素的指针, 可变数据, 可变指针
    const char *cpt = greeting;  // 不可变数据, 可变指针, 等价与char const *cpt
    // *cpt = 'a';  // error: read-only variable is not assignable
    char * const ptc = greeting;  // 可变数据, 不可变指针
    *ptc = 'a';  
    std::cout << greeting << std::endl;  //输出 aello
    // ptc = pt;  //error: read-only variable is not assignable
    const char * const p = greeting;  //数据指针都不可变
} 
```

<!--more-->

> 如果关键字const出现在星号左边, 表示被指物是常量(内存中的变量不可变), 如果关键字出现在星号右边, 表示指针本身是常量(不能再指向其他指针)


- 声明迭代器为const, 就想声明一个`T *const指针`, 其指针不可变值可以变, 如果不希望迭代器所指东西改变需要使用`const_iterator`
- 如果函数不会修改传你入的引用或指针类型参数, 该参数应声明为`const`.

> **两个成员函数如果只是常量性不同, 可以被重载**, 意思是const和非const成员函数, 被看做是两个不同的函数. const对象会首先调用const成员函数

```c
class Text {
public:
    Text(std::string str): text_(str) {}
    const char & operator[] (std::size_t pos) const { 
        std::cout << "call const method." << std::endl;
        return text_[pos]; 
    }
    char & operator[] (std::size_t pos) {
        std::cout << "call non-const method" << std::endl;
        return text_[pos];
    }
private:
    std::string text_;
};

int main(int argc, char *argv[]) {
    const Text test("hello");
    std::cout << test[1] << std::endl;  //调用const成员函数
    Text utext("world");
    std::cout << utext[1] << std::endl;  //调用非const成员函数

}
```

- 尽可能将函数声明为`const`. 访问函数应该总是`const`. 其他不会修改任何数据成员, 未调用非 const 函数, 不会返回数据成员非`const`指针或引用的函数也应该声明成 `const`.
- 如果数据成员在对象构造之后不再发生变化, 可将其定义为 const
- 关键字`mutable`可以使用, 但是在多线程中是不安全的, 使用时首先要考虑线程安全. mutable声明的数据成员即使在const成员函数中也是可以被更改的.


## Know what function C++ silently writes and calls

编译器可以暗自为class创建default构造函数, copy构造函数, copy assignment操作符, 以及析构函数

**禁用拷贝构造函数和赋值操作符的方案:**

- 仅声明拷贝构造函数和赋值操作符, 并声明为`private`. 但class成员函数和友元可能依然可以调用
- 设置一个禁用copying的`base class`, 所有不适用拷贝的函数都继承自这个基类

```c
class Uncopyable {
protected:
    Uncopyable() {}  //允许派生对象构造和析构
    ~Uncopyable() {} 
private:
    Uncopyable(const Uncopyable &);  // 阻止copying
    Uncopyable & operator=(const Uncopyable &);
};

class Base : private Uncopyable {  //private继承
    // do something, 禁用了copying
};
```

- 对于析构函数, 尽量不要在析构函数中引发异常, 如果发生错误, 最好强迫程序结束(abort)
- 不在构造函数和析构函数期间调用`virtual`函数
- 赋值运算符及复合运算符(如`+=, -=`)的重载都应该返回引用(支持链式调用), 对于包含指针数据成员的class, 赋值运算符的重载应该添加自我赋值的检测

```c
// 方案1: 进行自我赋值检测
    Text& operator=(const Text& rhs) {
        if (this == &rhs) {  //自我赋值检测两个对象地址是否相同
            return *this;
        }
        delete pt;  //pt是一个string类型的指针数据成员, 删除动态分配的空间
        pt = new string(*rhs.pt);  //重新开辟动态空间并初始化
        return *this;
    }
    
// 方案2: 不要先删除动态分配空间(更加安全)
    Text& operator=(const Text& rhs) {
        string* pstr = pt;  //创建一个新的指针保存动态分配空间的位置
        pt = new string(*rhs.pt);  //pt指向新开辟动态内存的位置
        delete pstr;
        return *this;
    }
// 方案3: copy and swap技术
思想: 将资源拷贝到对象副本中(保证原对象不会出错), 一切正常后, 对象副本与真正的对象交换
```

> 虚函数的运行机制!


## Use objects to manage resources

- 将资源放进管理对象(managing object)中被称为`资源取得时机便是初始化时机(RAII)`
- 管理对象(managing object)运用析构函数确保资源被释放
- `应用计数型智能指针(RSCP)`持续追踪共有多少对象指向某个资源, 但无法打破环状引用(智能指针智能监控局部对象使用状态). `垃圾收集机制(GC)`掌握全局的对象使用状态可以检测环状引用(或者称为循环引用)
- `STL中的auto_ptr和shared_ptr都不能用于动态分配数组(析构函数内为delete而不是delete[])`



## Treat class design as type design

- 新type的对象应该如何构造/析构(内存分配和释放)
- 对象初始化和对象赋值的区别(拷贝构造函数和赋值操作符实现的差别)
- pass-by-value和pass-by-reference的选择
- 错误检查, 边界检查和异常处理
- 新type的显式和隐式类型转换
- 那些成员函数应该private, 那些成员函数应该public


## Minimize casting

> 优良的C++代码很少使用转型

**转型动作:**

```
// old style casts, convert expression to type T
(T)expression
T(expression)

// new style cast(C++)
const_cast<T>(expression)
dynamic_cast<T>(expression)
reinterpret_cast<T>(expression)
static_cast<T>(expression)
```

- const_cast用来去除变量的常量性(去掉const属性)
- dynamic_cast用于`安全向下转型`(不同实现可能存在性能问题, 使用时需要慎重考虑)
- reinterpret_cast意图执行低级转换
- static_cast用于强迫隐式转换
- **注意:** 尽量使用C++ style cast, 不要使用旧式转型.


## Strive for exception-safe code

**异常安全(exception-safeJ)的条件:**

1. **不泄露任何资源**. 
2. **不允许数据破坏**

> 对于锁机制, 比较好的防止异常的方式是使用RAII

**异常安全函数提供三个保证之一:**

1. **基本承诺**: 如果异常被抛出, 程序内的任何事物仍然保持在有效状态下(对象和数据结构不会被破坏, 出现异常后, 程序可能处于任何状态)
2. **强烈保证**: 如果异常被抛出, 程序状态不改变(如果函数成功, 就完全成功, 如果函数失败, 程序会恢复到调用函数前的状态, 有点类似原子操作概念, 一般可以通过copy and swap策略实现)
3. **不抛异常保证**: 绝不抛出异常.


## Understand the ins and outs of inlining

1. inline函数减少调用函数招致的额外开销
2. 类内定义函数隐式声明为inline, 显式声明inline函数是在定义式前加上关键字inline
3. inline会增加目标代码大小(内存小的机器需要慎重考虑)
4. inline函数通常被定义于头文件, 大多数编译器inline为`编译期行为`. `inline是对编译器的神情, 编译器可以加以忽略`
5. 对于virtual函数(表示运行期才能确定实现), inline关键字失效

## Make sure public inheritance models "is-a"

> public inheritance意味"is-a"的关系(private和protected不是这种关系!)

在继承关系中, 使用名称遮掩的规则查找函数或者变量, 对于函数不会去匹配参数类型, 个数, 返回值.

使用using声明式, 来表示要使用函数或者变量所处的作用域. 
另外, `条款36提到, 绝不重新定义继承而来的non-virtual函数`

> 切记: `pure virtual(纯虚函数)`只具体指定接口继承, `impure virtual(虚函数)`具体指定接口集成并default实现继承, `non-virtual(非虚函数)`具体指定接口继承以及强制性实现继承

**区分动态绑定和静态绑定**:

假设有基类名为Base类, public继承的派生类为Derived类. Base类和Derived类分别有中有public非虚函数test()具体实现.

通过基类指针指向基类对象和通过基类指针指向派生类对象时, 调用test时, 都将调用基类的test(静态绑定). `由于指针为point to Base, 通过该指针调用的用于是指针对应类中的test` 


```c
#include <iostream>

class Base {
public:
    void test() {
        std::cout << "Base test" << std::endl;
    }
};

class Derived : public Base {
public:
    void test() {
        std::cout << "Derived test" << std::endl;
    }
};

int main(int argc, char** argv) {
    Base b;
    Derived d;
    Base *base = &b;
    Base *dbase = &d;
    base->test();
    dbase->test();
    return 0;
}
# 输出结果
Base test
Base test
```

如果将基类中的test改为虚函数, 此时则基类指针调用test发生了动态绑定.

```c
# 输出结果
Base test
Derived test
```

## Use private inheritance judiciously

- **铭记:** private不是一种is-a关系!
- private继承, 编译器不会自动将一个派生类对象转换为一个基类对象
- 由private基类集成来的所有成员, 在派生类中都会变成private


## Understand implicit interface and compile-time polymorhpism

> class和template都支持接口和多态

- 面向对象class以显式接口和运行期多态解决问题. 接口类型具体构造在源码中明确可见表示`显式接口`. 通常显式接口由函数的签名式(函数名称, 参数类型, 返回类型)构成.由类中的一些virtual函数可能导致函数的调用表现为`运行期多态`. 根据调用的类型来决定调用那个class中的函数
- template则注重隐式接口和编译器多态. `隐式接口`不基于签名式, 而是由有效表达式组成(和显式接口一样在编译期完成检查). 以不同的template参数具现化导致调用不同的函数, `具现行为`发生在编译期, 称为编译期多态


**派生类集成模板基类时(在模板具现化之前)无法看到基类中的具体实现的解决方案:**

1. 在调用基类函数动作前加上`this->`
2. 使用`using BaseClassName<>::some_member_method`, 请编译器假设函数位于基类中
3. 指出被调用的函数位于基类, `BaseClassName<T>::some_member_method` (被关闭virtual绑定行为)

## Use traits classes for information about types

STL中五种迭代器:

1. Input迭代器: 只能向前移动, 一次一步, 客户只能读取一次
2. Output迭代器: 只能向前移动, 一次一步, 客户端只能写数据一次
3. Forward迭代器: 只能向前移动, 一次一步, 可读可写
4. Bidirectional迭代器: 可以向前向后移动, 一次一步, 可读可写(list, set, map, multiset, multimap迭代器为这种)
5. Random access迭代器, 拥有以上功能外, 可以执行迭代器算术(一次执行多步移动, vector, deque, string为此类迭代器)

## Write placement delete if you write placement new

> 如果operator new 接受的参数除了size_t(需要分配大小)外还有其他参数, 这边上placement new


调用placement new后, 如果构造函数发生异常, 需要释放已分配的内存, 防止内存泄露, 此时在运行期间系统会寻找`参数个数和类型都与placement new`

```c
void operator new(std::size_t size, std::ostream& logSteam) throw(std::bad_alloc);  // placement new
void operator delete(void *pMemorty, std::ostream&  logStream) throw(); //placement delete

#调用
ClassName* pc = new (std::cerr) ClassName;
# 如果发生异常会调用placement delete, 否则delete pc会调用正常的operator delete
```

**类内专属的new会掩盖其他new(包括正常的global版本), 解决方案是专属new中::operator new(size)调用全局版本, 详见条款52**


## Miscellany

- 我曾经的老师, 告诉过我要把Warning当做Error来对待, 不要忽视Warning, 可能某一天这个Warning就会带来严重的Bug
- 推荐阅读侯捷先生的`<STL源码剖析>`, 让自己更加了解标准库
- 推荐了解更多Boost库

## 参考链接

- `Effective C++`
- [编译防火墙——C++的Pimpl惯用法解析](http://blog.csdn.net/lihao21/article/details/47610309)
