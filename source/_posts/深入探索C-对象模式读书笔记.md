---
title: 深入探索C++对象模式读书笔记
date: 2016-09-30 14:49:01
tags: C++
desc: C++ 对象模型 虚函数 虚表
---

> 什么是`C++对象模型`?
1. 语言中直接支持面向对象程序设计的部分.
2. 对于各种支持的底层实现机制.

## Object Lessons

1. C++封装并未增加布局成本, 数据成员内含在class object中(像struct), `成员函数不会出现在object中`, **非内敛函数只会诞生一个实例, 内联函数对每个使用者产生一个实例**
2. C++在布局及存取时间上主要的额外负担是由`virtual`引起的
    - virtual function支持动态绑定
    - virtual base class实现单一而被共享的基类实例, 多用于多继承中

<!--more-->

### C++ Object Model

1. 简单对象模式, object由很多slot组成, 每个slot(`指向member的指针`)指向一个memeber(包含数据成员和成员函数).
2. 表格驱动对象模型, 将members相关信息分别放到data member table(持有data member)和member function table(member function的指针)中, class object本身包含指向两个表格的指针
3. C++对象模型, 非静态数据成员配置在class object内, 静态数据成员存放在个别class object外.

C++对象模型之`虚函数`:

1. 每个`class(类)`产生一堆指向虚函数的指针, 放在virtual table(`vtbl`)中
2. 每个`object(实例)`包含一个指针(`vptr`), 指向相关的vitual table. `vptr的设置和重置由class的构造/析构/拷贝赋值操作符完成`. 每个class的`type_info object`(支持RTTI)也经由virtual table被指出来.


![](http://ww4.sinaimg.cn/large/ab508d3dgw1f7we2123j2j21ce0waq7v.jpg)


> 书中给出了一个struct和class结合使用的建议: 当要传递一个复杂的class object的全部或部分到C函数中, struct声明可以将数据封装起来, 并保证拥有与C兼容的空间布局.

C++程序设计模型支持三种programming paradigms(编程范式):

- 程序模型, 像C一样
- 抽象数据类型模型
- 面向对象模型

> 多态的主要用途是经由一个共同的接口来影响类型的封装, 这个接口通常被定义在一个抽象的base class中


## The Semantics of Constructors

### Default Constructor

四种情况会造成编译器必须为未声明构造函数的 classes 合成一个 default constructor

1. `带有Default Constructor的Member Class`. 如果class没有任何构造函数, 但包含一个`member object`, 而后者有默认构造函数, 编译器会为该clas合成一个default constructor(只是为了满足编译器的需求). 如果class有默认构造函数, 且包含`member object`, 则**编译器会扩张已存在的构造函数**.
2. `带有Default Constructor的Base Class`. 一个没有任何构造函数的class派生自一个带有default constructor的 base class, 这个派生类中的default constructor将会被合成.
3. `带有一个Virtual Function的class`. **默认构造函数正确的初始化每个class的vptr**. class声明一个虚函数会合成默认构造函数
4. `带有一个Virtual Base class的Class`, class派生自一个继承链, 其中有一个或者更多virtual base classes
    

带有虚函数类在编译器:

1. 编译期一个virtual function table会被编译器产生, 内放class 的virtual functions地址
2. 每个`class object`中, 一个额外的 pointer member(vptr) 被编译器合成出来, 内含相关与class vtbl的地址.


### Copy Constructor

```c
# 三个场景使用拷贝构造函数
class Base;
Base a;
Base b = a;  //1. 显示拷贝
void (Base x);  //2. object参数值传递
return b;  //3. 值传递的函数返回object
```

1. class未提供显式copy constructor时, 默认拷贝构造函数会把`内建或派生`的 data member 的值, 从某个object拷贝到另一个object上, 但不会拷贝其中的`member class object`


### Member Initialization List

以下情况必须使用成员初始化列表:

1. 当初始化一个 reference member 时
2. 当初始化一个 const member 时
3. 当调用一个 base class 的 constructor, 而它拥有一组参数时
4. 当调用一个 member class 的 constructor, 而它拥有一组参数时

- **初始化成员列表的顺序是由class中的members声明顺序决定的**
- 调用`member function`设定一个member的初值可行, 因为与此object 相关的 this指针已经被建构妥当

```c
X::X(int val) :
    i(xfoo(val)),  // xfoo必须是 member function
    j(val) {
}
```


## The Semantics of Data

### Data Member Layout

- 非静态数据成员在class object的排列顺序和其被声明的顺序一样
- `静态成员函数`不会放入对象布局中, 会被放入**data segment**

> C++标准对布局持放任态度.

### Data Member get/set

- `static data member`在class object中存取不会招致任何空间或执行时间上的额外负担(静态数据成员不在class object内部)
- `nonstatic data member`存放在 class object 内, 存取效率和 c struct member 或 nonderived class member 一样.


> 继承(非虚继承)不会不会增加空间和存取时间上的额外负担

菱形继承的解决方案是导入虚拟继承

![](http://ww3.sinaimg.cn/large/ab508d3djw1f80297j7bzj20ja0bqjs4.jpg)

虚继承的实现: class 如果内包含一个或多个 `virtual base class subobjects`, 将被分割成两部分: 一个不变区域和一个共享区域, 不变区域中的数据, 不管后继如何演化, 总有固定的offset用于直接存取, 共享区域只能被间接存取(在虚表中放置 virtual base class 的 offset)


## The Semantics of Function

### Nonstatic Member Functions

C++设计准则之一: 非静态成员函数至少必须和一般的非成员函数有相同的效率.

> 编译器将类的成员函数转换为非成员函数

转换步骤:

1. 改写函数signature, 增加一个额外的参数(this)
2. 将每一个`对 nonstatic data member 的存取操作`改为经过this指针存取
3. 将 member function 重新谢伟一个外部函数(对函数名特殊处理. `name mangling`)

### Virtual Member Functions

多态: 以 public base class的指针(或引用) 寻址出一个 derived class object.

- 一个类(不是对象)只有一个virtual table, 每个object被安插一个由编译器产生的指针(vptr).
- 派生的子类,会 overriding 一个可能存在的 base class virtual function实例

![](http://ww2.sinaimg.cn/large/ab508d3djw1f806zpmvegj21520y047e.jpg)

```c
class Derived : public Base1, public Base2 {};
# 两张虚表
vtbl_Derived;  // 主要虚表
vtbl_Base2_Derived;  // 次要虚表
```

**多重继承**情况下, 维护有多张虚表. 当Deriveed对象指定给Base1指针或Derived指针时, 被处理的virtual table 是主要表格`vtbl_Derived`, 当Deriveed对象指定给Base2时, 被处理的虚表为`vtbl_Base2_Derived`

![](http://ww2.sinaimg.cn/large/ab508d3djw1f807sjmgdfj214a108n6p.jpg)

### Static Member Function

> **没有this指针**

- 静态成员函数将被转换为一般的非静态成员函数调用
- 不能被声明为 const, volatile, virtual
- 不需要经由class object 调用

### Inline Functions

处理 inline函数的两个阶段:

1. 分析函数定义, 以决定函数的 `intrinsic inline ability`(我理解为天生的内联能力).
2. 真正的inline函数扩展操作是在调用的那一点上.


## 参考链接

- 深入探索C++对象模型

