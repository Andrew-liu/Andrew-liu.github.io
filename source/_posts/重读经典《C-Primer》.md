title: 重读经典《C++ Primer》
date: 2015-08-23 17:02:30
tags: C++
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


> 时隔一年, 重读C++ Primer这本`圣经`, 怀念去年这时基友们一起debug, 一起吃饭, 一起睡觉, 一起分享知识的那个夏天, 以这篇文章纪念我的好朋友, 希望有机会我们再聚在一起`吃酒撸串夜灯下诉过去与未来`, 同时以我微薄的知识向C++之父敬礼.

> 本文不会罗列C++基础语法, 只说明需要注意的地方, 所有需要注意的地方均为作者主观观点.

**如有任何错误之处, 欢迎斧正.**

# 编程风格

我认为学习一种语言, 一定要学习一种权威的编程风格指南, 就如python中的`PEP8`.

使用约定的风格, 可以减少协同工作的障碍, 建立程序猿之间的代码友谊. 虽然朋友说要对别人的代码宽容, 而我认为这就像纵容别人犯罪一样, 遵守一定的代码风格, 能瞬间拉近程序猿之间的距离, 增加代码可读性何乐而不为呢? `我喜欢看开源代码的原因之一就是很多著名的开源项目优雅的编程风格` 

<!--more-->

简要的罗列一下建议遵守的编程风格(谷歌风格):

- 所有头文件都应该使用 #define 防止头文件被多重包含, 命名格式当是: `<PROJECT>_<PATH>_<FILE>_H_`
- 当只有数据时使用 `struct`, 其它一概使用`class`
- 在类中使用特定的声明顺序: `public:` 在 `private:` 之前, 成员函数在数据成员 (变量) 前
- 整数用`0`, 实数用`0.0`, 指针用`NULL`, 字符(串)用 `'\0'`.
- 类, 结构体, 类型定义 (typedef), 枚举的每个单词首字母均大写, 不包含下划线(`大驼峰命名`)
- 变量名一律小写, 单词之间用下划线连接. `类的成员变量以下划线结尾`
- 常量和枚举类型命名在`名称前加k`: kDaysInAWeek
- 常规函数使用大小写混合(`大驼峰命名`), get和set函数则要求与变量名匹配(`set函数前加set前缀`) 
- 逗号后添加空格
- 不建议使用`using namesapce std`


更多细节参考[Google 开源项目风格指南](http://zh-google-styleguide.readthedocs.org/en/latest/)


# 基础

**编译与执行:**

1. 预处理阶段: 根据字符#开头的命令, 修改原始C程序
2. 编译阶段: 将文本文件翻译成汇编程序
3. 汇编阶段: 汇编器将编译程序翻译成`机器语言指令`(机器可识别), 并打包成可重定位目标程序
4. 链接阶段: 将调用函数目标文件合并到程序中, 形成可执行目标文件

```c++
//使用GUN编译器g++, -o选项将输入写入文件, 即用来存放可执行文件
$ g++ chapter1.cc -o chapter
```


**变量:**

变量的作用我认为有以下几点:

- 给一定大小的内存命名, 方便使用和增加可读性
- 通过变量的类型, 来决定到底访问几个字节长的内存
- 通过变量的来决定如何解释所读取的内存的数据(如有符号数和无符号数的解释不同)

**左值与右值:**

- 左值可以出现在赋值语句的左边
- 右值只能出现在赋值的右边, 不能出现在赋值语句的左边

**const关键字:**

1. 通过指定const变量为extern, 可以在整个程序中访问const对象
2. const引用是指向const对象的引用, 对象可读不可写
3. 指向const对象的指针, 定义时不需要初始化, 可以对指针重新赋值(修改其中保存的内存地址, 指向其他对象), 但所指向对象中的值不能修改(内存中保存的值不能修改)
4. const指针, const指针的值(保存的内存地址的值不能修改)不能修改, 也就是不能使const指针指向其他对象
5. 指向const对象的const指针, 既不能修改指针所指向的对象值(内存地址中保存的值), 也不能修改指针的指向(指针中保存的内存地址)

```c
// file1.cc
extern const int buf_size = 10;
// file2.cc
extern const int buf_size; //使用file1.cc中的buf_size

//指向const对象的指针
const double *cptr;
//const指针
int err_numb = 2;
int *const cur_err = &err_numb; 
```


**预处理器:**

- 使用预处理器变量避免头文件的多重包含

**常用格式:**

```c
#ifndef _XXX_H_
#define _XXX_H_

...

#endif /* _XXX_H_ */
```


**指针:**

- 指针保存的是对象的`地址`
- 数组名会自动转换为`指向数组第一个元素`的指针.
- 数组的下标访问数组时实际上是使用下标访问指针, `指针是数组的迭代器`
- 当类中有指针数据成员, 不能使用系统自带的拷贝构造函数(系统默认导致浅拷贝, 两个对象指针指向同一块动态分配的内存)/赋值函数(=操作符重载), `请自定义`(深拷贝)



**自增/自减操作符**

- 前自增操作加1后返回加1的结果
- 后自增操作保存操作数原来的值, 返回未加1之前的值作为操作的结构
- 自减操作符类似与自增操作符


**sizeof操作符**

> 注意`sizeof`是操作符, 用于获得类型的长度

- 对数组做sizeof操作等效于将对其元素类型做sizeof操作的结果乘上数组元素的个数.
- 对指针做sizeof将`返回存放指针`所需的内存大小
- 对引用做sizeof将`返回存放此引用`类型对象所需所需的内存大小


> **switch**语句执行匹配的case标号相关联的语句后, 会`跨越case边界`继续执行其他语句, 直到switch结束或者遇到break.


**复制传参(pass by value)和引用/指针传参(pass by reference):**

- 复制传参无法改变传入实参的值
- 复制传参增加了时间和存储空间的开销, `尽量减少pass by value`
- 引用传参相当于传指针
- 返回值也要尽量传递引用(不要返回局部变量的引用)


**static对象:**

> 类内staitc数据成员不属于某个对象(类内声明, 类外定义). 类内static函数没有this指针(用于处理staitc数据成员)

static成员必须在class定义式之外被定义(除非他们是const并且是整型)

```
class Test {
public:
    Test(const int p): price(p) { std::cout << "constructor Test." << std::endl; }
    void setRate(const double r) { this->rate = r; }
    double getRate() { return rate; };
private:
    static double rate;  //declare
    int price;
};
double Test::rate = 0.53;  //define

int main(int argc, char *argv[]) {
    Test t(10);
    Test t1(20);
    std::cout << t.getRate() << std::endl;  
    std::cout << t1.getRate() << std::endl;
    t.setRate(2.22);
    std::cout << t.getRate() << std::endl;
}
```

一旦创建static对象被创建, 在程序结束前不会被销毁. 常用于生命周期跨越多个函数调用的对象


**inline函数:**

- 普通函数被调用: 调用前保存寄存器, 返回时恢复上下文, 复制实参, 程序还必须转向一个新的位置执行
- inline函数在编译时被展开, 从而消除额外的函数执行开销, 常用于小操作函数.`内联函数要在头文件中定义`


**重载函数:**

1. 确认候选函数(`C++名字查找发生在类型检查之前`)
2. 检查形参个数和形参类型匹配问题


**函数指针:**

重点理解: 函数指针是`指向函数的指针`
**直接使用函数名等效于在函数名上取地址操作符**

```
// pf是一个指针, *表明了pf的指针身份, pf的类型为bool (const string &, const string &)
bool (*pf)(const string &, const string &);
//类比与普通变量指针, *表示ps是指针, ps的类型为const string
const string *ps;
// typedef简化函数指针定义 cmpFun等价于bool * (const string &, const string &)
typedef bool (*cmpFun)(const string &, const string &);
cmpFun pf;
//typedef简化普通变量指针 pstr等价于 const string *
typedef const string *p_str;
pstr ps;
```

## Stack和Heap

Stack是存在于某作用域的一块内存空间
Heap是由操作系统提供的一块全局内存空间(可动态分配获得此类空间)


## namespace

**标准库中所有文件被包裹在std命名空间中.**

命名空间可以是不连续的

```
namespace myname {
    ...
}  // 不以分号结束
```


## 异常

异常通过`throw`抛出对象引发的.**异常可以传递给给非引用形参任意类型的对象**

1. 通过栈展开(stack unwinding), 沿嵌套函数调用链继续向上, 直至为异常找到一个用于处理异常的catch语句
2. 捕获所有异常的catch子句形式为`(...)`
3. exception类型所定义的唯一操作是what虚函数

# 类

> 类定义了一个新的类型和新的作用域, **切记类定义以分号结束**

> `struct`和`class`的唯一差别在于默认访问级别上, struct的成员默认为public, class成员默认为private


- **类静态成员static**: 静态数据成员被`类`的所有对象所共享, 包括该类派生类的对象, 也就是说, 静态数据成员属于类, 而不属于某个对象. static成员函数没有this指针, 不能被声明为虚函数.
- **隐式形参this**: 类中每个成员函数都有一个`额外的、隐含的形参(this, const成员函数时, this的类型为const class_type *const this)`将该成员函数与调用该函数的类对象绑定在一起. `形参this初始化为调用函数的对象的地址`
- **常量成员函数**: 在成员函数形参表后声明const, 用来表明隐式形参this的类型为`const class_type *`
- **构造函数**: 与类同名且没有返回值, 一个类可以有多个构造函数, 构造函数不能声明为const. `注意:其中的构造函数的初始化列表有顺序`
- **const成员函数**: 对于一个不会改变数据的函数, 果断加上`const关键字`, 表示该函数不允许修改类的数据成员
- **虚函数**: 虚函数希望派生类对此函数进行override, 纯虚函数要求派生类必须override这个函数. `注意: 通过基类的引用或指针调用虚函数才能引发动态绑定`

> 理解初始化列表, 初始化列表初始化数据成员(注意const对象或引用类型只能初始化不能赋值), 没有初始化列表的的构造函数在函数体中对数据成员`赋值`. `成员被初始化的顺序就是定义成员的顺序`

```
// 链式编程
Screen & Screen::move(char c) {
    contents[cursor] = c;
    return *this;  // this是一个指针, 解引用后是一个类类型Screen
}
```


## 构造函数和析构函数的顺序

**直接上代码会比较清晰**

```c
class Foo {
public:
    Foo() { std::cout << "Foo default constructor." << std::endl; }
    Foo(const Foo &foo) { std::cout << "Foo copy constructor." << std::endl; }
    ~Foo() { std::cout << "Foo deconstructor." << std::endl;}
};

class Bar {
public:
    Bar() { std::cout << "Bar default constructor." << std::endl; }
    Bar(const Bar &bar) { std::cout << "Bar copy constructor." << std::endl; }
    ~Bar() { std::cout << "Bar deconstructor." << std::endl;}
};

class Yes {
public:
    Yes() { std::cout << "Yes default constructor." << std::endl; }
    ~Yes() { std::cout << "Yes deconstructor." << std::endl; }
};

class Base {
public:
    Base() { std::cout << "Base constructor." << std::endl; }
    ~Base() { std::cout << "Base deconstructor." << std::endl; }
private:
    Foo foo_;
};

class Derived : public Base {
public: 
    Derived() { std::cout << "Derived constructor." << std::endl; }
    Derived(const Bar &bar, const Yes &yes);
    Derived(const Yes &yes, const Bar &bar);
    ~Derived() { std::cout << "Derived deconstructor." << std::endl;}
private:
    Bar bar_;
    Yes yes_;
};

Derived::Derived(const Bar &bar, const Yes &yes) {
    std::cout << "Derived argument: (bar, yes) constructor." << std::endl;
}

Derived::Derived(const Yes &yes, const Bar &bar) {
    std::cout << "Derived argument (yes, bar) constructor." << std::endl;
}

int main(int argc, char *argv[]) {
    std::cout << "create simple obejct foo and bar." << std::endl;
    Foo foo;
    Bar bar;
    Yes yes;
    std::cout << "create Base class " << std::endl;
    Base base;
    std::cout << "case 1 : (default constructor) " << std::endl;
    Derived derived1;
    std::cout << "case 2 : (argument bar, yes)" << std::endl;
    Derived derived2(bar, yes);
    std::cout << "case 3 : (argument yes, bar)" << std::endl;
    Derived derived3(yes, bar);
}
```


运行结果:

```
// 创建三个简单的对象
create simple obejct foo and bar.
Foo default constructor.
Bar default constructor.
Yes default constructor.

// 创建基类对象
create Base class
Foo default constructor.  //先对成员变量初始化
Base constructor. // 调用基类构造函数

// Derived调用默认构造函数
case 1 : (default constructor)
Foo default constructor.  //先调用基类构造函数
Base constructor.
Bar default constructor.  //自身成员变量初始化
Yes default constructor.
Derived constructor.  //调用默认构造函数

// 调用以bar, yes为参数的构造函数
case 2 : (argument bar, yes)
Foo default constructor.
Base constructor.
Bar default constructor.  //注意bar, yes构造的顺序
Yes default constructor.
Derived argument: (bar, yes) constructor.

//调用以yes, bar为参数的构造函数
case 3 : (argument yes, bar)
Foo default constructor.
Base constructor.
Bar default constructor.  //注意bar, yes构造的顺序
Yes default constructor.
Derived argument (yes, bar) constructor.

//析构过程(构造过程的逆)
Derived deconstructor.  // case3的析构
Yes deconstructor.
Bar deconstructor.
Base deconstructor.
Foo deconstructor.

Derived deconstructor.  // case2的析构
Yes deconstructor.
Bar deconstructor.
Base deconstructor.
Foo deconstructor.

Derived deconstructor.  // case1的析构
Yes deconstructor.
Bar deconstructor.
Base deconstructor.
Foo deconstructor.

// Base的析构
Base deconstructor.
Foo deconstructor.

//三个简单对象的析构
Yes deconstructor.
Bar deconstructor.
Foo deconstructor.
```

- 类先初始化成员变量, 然后执行构造函数内部逻辑
- 成员对象初始化的次序完全不受它们在初始化表中次序的影响, 只由成员对象在类中声明的次序决定
- 析构函数的调用顺序和构造函数调用顺序相反

参考[浅出C++对象模型——理解构造函数、析构函数执行顺序](http://blog.csdn.net/gaoyingju/article/details/8790233)

## 复制控制

> 具有指针成员的类一般需要定义自己的复制控制(防止浅拷贝), 复制构造或赋值操作符应该显式使用基类的复制构造或赋值操作符

- **复制构造函数**: 形参通常为const引用, 一般不设置为`explicit`. `禁用复制需要将复制构造函数声明为private`. (含有指针数据成员时, 避免浅拷贝应该定义复制构造函数)
- **赋值操作符**: 类内赋值操作符重载, 包含隐式this形参, 右操作数一般为const引用, 返回值一般应该为引用.
- **析构函数**: 资源回收, 尤其是指针所指向的动态分配的内存, 析构函数不可重载

```c
拷贝赋值函数(赋值操作符重载): 

1. 检测值是否自我赋值(self assignment, 地址比较 this == &object)
2. 左值内存delete清空
3. 分配与右值相同大小内存
4. 赋值到左值
```

**操作符重载原则**

- 赋值(返回对*this的引用), 下标, 调用([])和成员访问箭头必须定义为成员函数
- 符合操作符通常定义为成员函数
- 自增, 自减, 解引用操作符通常定义为类成员函数(改变对象状态或与类型紧密相关)
- 算术, 相等, 关系, 位, 流操作符一般定义为普通非成员函数(设置为友元)



## 对象的创建


**C++提供两种方法分配和释放未构造的原始内存:**

1. allocator类, 提供可感知类型的内存分配
2. 标准库中的operator new和operator delete分配和释放需要的大小的原始的、未类型化的内存


```c++
1. ClassName object(para);

// 使用new(先分配内存malloc, 指针类型转换 然后调用构造函数)后, 需要delete(先调用析构函数, 再释放内存free)掉分配的堆内存, 防止内存泄漏
// 该表达式调用名为operator new的标准库函数分配足够大的原始的未类型化的内存
2. ClassName *object = new ClassName(param);

// 当使用delete表示删除动态分配内存时, 首先对object指向的对象析构, 然后调用operator delete的标准库函数释放该对象所用的内存, operator delete不会调用析构函数
3. delete object;

//placement new不分配内存, 而是使用已分配但未构造内存的初始化一个对象(接受一个指针)
4. new (place_address) type  // place_address为指针
```

[关于C++内存分配的new, operator new, placement new](http://beatrice7.tk/2015/06/06/New/)


- C++创建对象时`仅分配用于保存数据成员`的堆空间, 成员函数没有单独的空间
- C++用new创建对象时返回一个`对象指针`, object指向一个ClassName的对象, `C++分配给object的仅仅是存放指针值的空间`, 并且用new 动态创建的对象必须用delete来撤销该对象(只有delete对象才会调用其析构函数)
- 相同class的各个对象互为友元(friend)


## 类模版

> 泛型编程是以独立于任何特定类型的方式编写代码

 
**模板形参可以是类型形参, 也可以是非类型形参**

模板本身并不是一种类型, 当编译器看到模板定义的时候, 不立即产生代码, `只有在看到用到模板时, 编译器才会对模板进行实例化`


- 在类本身的作用域内, 可以使用类模板的非限定名
- 类外定义的类模板成员函数, 必须以关键字template开头, 后接类的模板形参表, 必须指出是那个类的成员并包含模板形参
- 类可以拥有本身为类末班或者函数模板的成员(成员模板). 另外, 此类成员模板定义在类外部时, 需要包含两个模板的形参表
- 模板特化, 我认为是对一些特殊模板形参进行实现(比如模板形参中包含指针的时候)


```c
template <class Type> 
class Queue {
public:
    Queue() {}; // 可以使用Queue<Type>, 由编译器推断
private:
    QueueItem<Type> *head;  // 非类作用域空间必须显式使用模板形参
    void destory();
};

template <class Type> 
void Queue<Type>::destory() {
    // something
}
```

# STL

**输出/输出流**:

- std::cout, 结果是左操作数的值, `输出操作返回的是输出流本身`
- std::cin, 类似于`std::cout`, 输入操作符返回器做操作数作为结果


##顺序容器

> **顺序容器**：将单一类型元素聚集起来成为容器, 然后根据位置来存储和访问这些元素. 顺序容器包括`vector, list, deque`, 顺序容器的适配器`stack(基于deque), queue(基于deque), priority_queue(基于vector)`

**vector对象动态增长:**

vector的元素`连续存储`. 其中size()函数统计vector已有元素个数, capacity()指在vector必须重新分配存储空间之前可存储的元素个数.可见vector分配存储空间的策略是`增幅小于1时取1, 之后每达到2的次方时倍增(变为原来容量的2倍)`


```c++
// 测试程序
void TestIncrease(std::vector<std::string> &vec) {
    std::cout << "size: " << vec.size() << std::endl;
    std::cout << "capacity: " << vec.capacity() << std::endl;
}

void TestVector() {
    std::vector<std::string> vec;
    TestIncrease(vec);
    for(std::vector<std::string>::size_type ix = 0;
        ix != 24; ++ix) {
        vec.push_back("hello");
        TestIncrease(vec);
    }
}
// 测试结果, 只保留重要部分
size: 0
capacity: 0
size: 1
capacity: 1
size: 2
capacity: 2
...
size: 8
capacity: 8
size: 9
capacity: 16
...
size: 16
capacity: 16
size: 17
capacity: 32
...
size: 24
capacity: 32
```

**const_iterator和const的iterator:**

> 迭代器可以理解为指针.

- `const_iterator`创建的对象, 自身的值可以改变, 但不能改变其指向的元素的值, 对对象解引用返回的是一个const值
- 声明const的迭代器时, 必须初始化, 初始化后, 迭代器自身不可再变, 迭代所指向的元素值可以改变(`这里const修饰的是迭代器`)


## 关联容器

关联容器和顺序容器的本质区别: **关联容器通过key存储和读取元素, 顺序容器通过位置存储和访问容器**


- map通过key的`小于关系`排序, 自定义数据结构应该重载`<`操作符. map迭代器解引用产生pair对象

> 关联容器`map, set`


# 参考链接及书目


- <C++ Primer> 第四版
- [What is the use of “using namespace std”?](http://stackoverflow.com/questions/18914106/what-is-the-use-of-using-namespace-std)
- [C/C++ 预处理器参考](https://msdn.microsoft.com/zh-cn/library/y4skk93w.aspx)
- [Super fast C++ logging library](https://github.com/gabime/spdlog)
- C++预处理和预定义
- [预处理指令 (Preprocessor Directives)](http://www.prglab.com/cms/pages/c-tutorial/advanced-concepts/preprocessor-directives.php)

