title: Java作死笔记
date: 2015-03-15 12:38:27
tags: MapReduce
---


本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


> 本来还想在校期间任性的不学习Java, C/C++/Python一直用到毕业呢, 结果`天不遂人愿`, 选修了MapReduce不得不学习Java了, 完全面向对象语言的学习还有一点小期待的.
> 这篇博文用来记录Java学习过程, 用来学习复习巩固之用.



#1. Java基本程序设计结构
---

> Java应用程序中全部内容都必须放置在类中

- 类名以首字母大写的驼峰命名法
- 源代码的文件名必须与公共类的名字相同, 并且以`.java`作为后缀
- java中所有函数都属于某个类的方法(java中main方法必须有一个外壳类)

```java
javac FirstSample.java  //java编译器将源代码编译成字节码
java FirstSample  //java虚拟机从指定类中的main方法开始执行
```

<!--more-->

> java中单行注释和多行注释的方式和C++相同


另外还有一种文档注释(文档注释一般出现在源文件顶部)

```
/*
 * This is the doc comment
 * 
 */
```



> java是一种强类型语言, 且大小写敏感

```
//变量的定义和初始化
int vacationDays = 12;
double salary = 6500.2;
//常量的定义使用关键词final, 使用static final关键字设置类常量
final double CONSTANT_CASE = 2.54;
```

**运算符**

- 加减乘除运算符
- 自增, 自减运算符
- 关系运算符
- 位运算符
**这些运算符与C++的定义基本相同**


二元操作的操作数的转换`double > float > long > int`

**枚举类型**

```java
enum Size{
    SMALL,
    MEDIUM,
    LARGE,
    EXTRA_LARGE
};
```


**字符串**

```
String e = "";
String first = "Hello";
String second = first.substring(0, 3);  //提取字符串中子串
String message = first + second;  //字符串拼接操作
first.equals(second);  //字符串比较是否相等
first.length();  //计算字符串的长度
if(first == null)  //检查字符串是否为null
```

> 当一个字符串和一个非字符串进行拼接时, 后者被转换成字符串, Java中String类对象称为不可变字符串, 对字符串的操作会生成一个新的字符串, `不可变字符串的优点是编译器可以让字符串`


**输入输出**

```java
Scanner in = new Scanner(System.in);
System.out.println("What's your name?");
String name = in.nextLine();  //输入
```

**文件的输入和输出**

```java
import java.io.PrintWriter;
import java.nio.file.Paths;
        Scanner myFile = new Scanner(Paths.get("test.txt")); //读取文件
        PrintWriter out = new PrintWriter("test.txt");  //写入文件
```

控制流程语句的写法与C++基本相同

Java还提供了一种`带标签的break语句`, 用于跳出多重嵌套的循环语句, 标签必须放在希望跳出的最外层循环之前, 并且必须紧跟一个冒号.


**数组**

- 声明数组变量时, 需要指出数组类型和数组变量的名字
- 创建数字数组, 所有元素初始化为0, boolean数组的元素会初始化为false, 对象数组的元素则初始化为null
- 数组大小在创建后不可变

```java
int[] a = new int[100]
```

**for each循环**

```java
        int[] a = new int[10];
        int[] b = {2, 3, 5, 7};
        for(int element : a)
        {
            System.out.println(element);
        }
```

**命令行参数**

- 在Java中程序名没有被存储在args数组中


#2. 对象与类
---

> 识别类的简单规则是在分析问题的过程中寻找名词, 而方法对应着动词.

> 所有的Java对象都存储在堆中.

- 对实例做出修改的方法叫做更改器方法
- 仅访问实例域而不修改的方法叫访问器方法(C++中带const后缀的方法)
- `源文件名必须与public类的名字相匹配, 并且只能有一个public公共类, 可以有任意数目的非共有类`
- 类中包含构造器(构造函数)和方法, 构造器随着new操作符的执行被调用
- java中所有方法必须在类内定义

```
new Date();  //构造一个新对象
```

**构造器**

1. 构造器与类同名
2. 每个类可以有`一个以上`的构造器
3. 构造器可以有0个或者多个参数
4. 构造器没有返回值
5. 构造器伴随new操作符调用

```
import java.util.*;
˙
class Employee {
    private String name;
    private double salary;
    private Date hireDay;
    public Employee(String n, double s, int year, int month, int day) {  //构造器
        name = n;
        salary = s;
        GregorianCalendar calendar = new GregorianCalendar(year, month - 1, day); // 月份从零开始
        hireDay = calendar.getTime();
    }
    public String getName() {
        return name;
    }
    public double getSalary() {
        return salary;
    }
    public Date getHireDay() {
        return hireDay;
    }
    public void raiseSalary(double byPercent) {
        double raise = salary * byPercent / 100;
        salary += raise;
    }
}
```

**类中方法包含一个隐式参数,是出现在方法名前的类对象,可以用类似C++的this写法表示隐式参数**

> 注意对于可变数据域(有方法改变其状态), 不要编写返回引用的访问器, 首先应对其进行clone,即`返回一个可变数据域的拷贝, 应该使用clone`

##2.1. 类的访问权限

> 类内方法可以访问所属类的私有特性

- `final`实例域表示在对象构造以后, 这个实例域不可变, 多应用于不可变类的域
- `static`, 当修饰实例域, 表示类的所有势力都共享一个实例域. 静态方法是一种不能向对象实施操作的方法(没有隐式参数)
    1. 一种方法不需要访问对象状态
    2. 一种方法只需要访问类的静态域

- 标记public的部分可以被任意的类使用
- 标记private的部分只能被定义它们的类使用
    

##2.2. 初始化数据域方法

1. 在构造器中设置值
2. 在声明中赋值
3. 初始化块(先运行初始化快, 然后才运行构造器的主体部分)

##2.3. 对象析构

> 可以为类添加finalize方法, 在垃圾回收器清除对象之前调用, `不要依赖这个方法, 因为很难知道何时调用`


##2.4. 包

> 使用包的主要原因是确保类名的唯一性

访问另一个包中的共有类:

1. 每个类名之前添加完整的包名
2. 使用import语句

**将类放入包中,使用package关键词**


##2.5. 注释

- 普通行注释`//`
- 多行注释`/* ... */`
- 文档注释`/** ... */`
- 方法注释
    - `@param`变量描述
    - `@return`返回值描述
    - `@throws`异常描述
- 通用注释
    - `@author`作者姓名
    - `@version`版本条目
    - `@since`对引入特性的版本进行描述
    - `@deprecated`文本中取代(过时)的建议
    - `@see`添加超级链接


##2.6. OOP技巧

1. 保持数据私有(不要破坏封装性)
2. 对数据初始化
3. 不要在类中使用过多的基本类型
4. 不是所有类都需要域访问器和域修改器
5. 将职能过多的类分解
6. 类名和方法名要体验他们的职责

#3. 继承
---

> Java中所有继承都是公有继承, 没有C++中似有私有继承和保护继承(`is-a`)

- 一个对象变量可以指示多种实际类型的现象称为多态. 在运行时能够自动选择调用那种方法的现象叫做动态绑定.
- Java中动态绑定是默认的处理方式
- Java不支持多继承
- Java中`对象变量是多态的`
- `不能将一个父类的引用赋值给子类变量`

> 阻止继承, 在定义类时, 使用final关键字, final声明的类方法, 子类不能覆盖

```
// 继承的写法
class Manager extends Employee {
    private double bonus;
    public Manager(String n, double s, int year, int month, int day) {
        super(n, s, year, month, day);
        bonus = 0;
    }
    public void setBonus(double b) {  //子类新增函数
        bonus = b;
    }
    public double getSalary() {  //重写父类的方法
        double baseSalary = super.getSalary();  //调用父类的方法
        return baseSalary + bonus;
    }
}
```

##3.1. 动态绑定

调用对象方法的执行过程:
1. 编译器查看对象的声明类型和方法名
2. 编译器查看调用方法时提供的参数类型(`重载解析`)
3. private, static, final方法编译器可以准确知道调用那个(`静态绑定`)
4. 程序运行时, 采用动态绑定调用方法时, 虚拟机一定调用与x所引用对象的实际类型最合适的那个类的方法


**类的强制类型转换**

- 只能在继承层次内进行类型转换
- 在将父类转换成子类前, 应该使用instanceof进行检查

```java
        if (staff[1] instanceof Manager) {
            boss = (Manager)staff[1];
        }
```


> `abstract`关键字, 包含一个或多个抽象方法的类必须声明为抽象的.(抽象类中可以定义具体方法), 抽象类不能被实例化


##3.2. 枚举类

```
public enum Size {
    SMALL, MEDIUM, LARGE, EXTRA_LARGE
};
```

##3.3. 继承设计技巧

1. 将公共操作和域放在基类中
2. 不要使用protected域, 它会破坏封装性
3. 使用继承实现`is-a`关系
4. 除非所有继承的方法都有意义, 否则不要使用继承
5. 在覆盖方法时, 不要改变其功能
6. 使用动态, 而不是类型信息
7. `不要过多的使用反射`




#4. 接口和内部类
---

> 接口的所有方法自动属于public, 接口不能含有实例域, 也不能在接口中实现方法, 接口可以看做没有实例域的抽象类. `接口不是类`


类实现接口的步骤:

1. 将类声明为实现给定的接口
2. 将接口中的所有方法进行定义

```java
public interface Comparable {  // 接口
    int comparaTo(Object other);
}
class Employee implements Comparable<Employee> {  //类实现接口
    public int comparaTo(Employee other) {
        return Double.compare(salary, othre.salary);
    }
}
```

> 为什么Java有了抽象类还要引入接口概念

- 每个类只能扩展于一个类
- 每个类可以实现多个接口

```
class Employee extends Comparable, Cloneable {
}
```

##4.1. 对象克隆

*浅拷贝和深拷贝问题*

- 浅拷贝中原始变量与拷贝变量引用同一个对象, 指向同一块内存
- 深拷贝在我的理解看来, 就是将原来的对象, 重新内存开辟一块中间, 生成完全相同的对象的复制.


```
class Employee extends Cloneable {
    public Employee clone() throws CloneNotSupportedException {
        Employee cloned = (Employee) super.clone();  // 调用Object的clone方法
        cloned.hireDay = (Date) hireDay.clone();  // 克隆可变域
        return  cloned;
    }
}
```



#5. 异常
---

> Java中, 异常对象都是派生于Throwable类的实例

设计Java程序时, 需要关注Exception层次结构.


抛出异常的情况: 

1. 调用一个抛出已检查异常的方法
2. 程序运行过程中发生错误, 并且利用throw语句抛出一个已检查异常.
3. 程序出现错误
4. Java虚拟机和运行时库出现的内部错误


- 如果一个方法可能抛出多个已检查异常, 必须在方法首部列出所有异常类, 逗号隔开

> 抛出异常使用`throw`关键字

```
// 自定义异常类
class FileFormatException extends IOException {
    public FileFormatException() {}
    public FileFormatException(String gripe) {
        super(gripe);
    }
}
```


    
##5.1. 捕获异常

> 捕获异常, 使用`try/catch`语句

 - 如果try语句块中的代码抛出一个catch语句说明的异常类
 - 程序将跳过try语句其他代码, 执行catch语句中的处理代码
 - 如果try语句块无异常, 则跳过catch语句继续执行
 - 使用finally进行本地资源回收(`不管异常是否被捕获, finally中语句都会被执行`)
 

##5.2. 使用异常的技巧

1. 异常处理不能代替简单的测试
2. 不要过分的细化异常
3. 利用异常层次结构
4. 善于传递异常

##5.3. 断言

> 默认情况下, 是禁用断言的, 运行程序使用选项 -enableassertions或者-ea启用

```
// 两种形式
assert 条件;
assert 条件 : 表达式;  // 表达式会传入AssertionError构造器, 转换成消息字符串.
```
