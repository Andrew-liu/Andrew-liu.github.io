title: 非零基础JavaScript简易指南
date: 2015-02-16 20:33:51
tags: JavaScript
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

> 写作意图: 一直通过一些视频或者网页零碎的学习JavaScript, 整个语言一直难以形成系统, 所以希望利用一些时间系统的学习一下JavaScript语言.

写作目标:

- 希望构建JavaScript语言体系, 将零碎知识点系统化
- 希望以简单的语言, 能够使有编程基础的人快速入门JavaScript

> 注意: 本书并不针对零基础者, 很多基础内容会一笔带过, `本书大部分内容基于<JavaScript权威指南>进行总结`.

----

#0. JavaScript概述

> JavaScript是一门高端的, 动态的, 弱类型的编程语言

- 语言区分大小写
- 用`unicode`编码编写
- 注释使用`// 单行注释`或者`/* 多行注释 */`, 和C++相同
- 以`;`作为行间分隔符可选, 可以使用C++风格或者Python风格

<!--more-->

#1. 类型/值和变量
---

> JavaScript的数据类型分为: 原始类型和对象类型.

- 原始类型包括数字类型, 字符串类型, 布尔类型, null和undefined(`最后两种是特殊的原始值`)
- 对象类型包括普通对象, 具有特有行为的array数组对象和函数对象

> JavaScript的数据类型也可分为: 可变类型和不可变类型

- 可变类型包括数组和对象(可变类型的比较是引用的比较)
- 不可变类型包括数字类型, 字符串类型, 布尔类型, null和undefined(不可变类型的比较为值的比较)

```js
var a = [], b = [];
a === b;  // false

var a = [];
var b = a;
b[0] = 1;
a[0];
a === b;  // true
```

> JavaScript变量是无类型的, 可以被赋予任何类型值


**数字**

**JavaScript无整数和浮点数之分, 所有数字都用浮点数(IEEE754)表示**

JavaScript中算数运算在溢出, 下溢或被零整除时不会报错, 溢出或者被零整除返回`Infinity`或`-Infinity`, 下溢返回0和-0,


```js
2  
3.14
4.15e23 // 4.15 * 10^23
```

> 判断变量x是否是NaN(非数字值), 若x != x为true, 则x为NaN

**字符串**

```js
""
'string'
"JavaScript"
'name = "Andrew"'
```

- 支持字符串`+`拼接操作
- 字符串length属性, str.length
- 支持定位, 取子串, 切片, 分割, 替换等字符串操作

> 字符串本身是不可变的!

**布尔值**

true和false

- undefined, null, 0, -0, NaN, ""转换为false
- 其他值包括对象(数组)会转换成true


**类型转换**

**显式类型转换**

```js
Number("3");  // 转换为数字3
String(false);  // 转换为字符串"false"
Boolean([]);  // 转换为布尔值true
Object(3);  //  转换为对象Number {[[PrimitiveValue]]: 3}
```

**隐式类型转换**

- `+`运算一个操作数是字符串, 会将另一个操作数转换为字符串
- `!`运算会把操作数转换为布尔值

**变量声明**

使用var关键字声明

```js
var i = 0;
var str = "hello";
var first = function() {}
```


**作用域**

- 函数体内, 同名局部变量的优先级高于全局变量(局部变量和全局变量同名不是好的编程习惯)
- JavaScript中没有块级作用域, 使用函数作用域(函数内声明在整个函数体都是可见的)


#2. 表达式和运算符
---

```js
var matrix = [1, 2, 3];  // 数组初始化
var p = { x: 2.3, y: 1.8};  // 对象初始化
var square = function {  // 函数直接量
    return x * x;
}
// 属性访问.语法和[]语法
// 调用表达式()语法, 如函数调用
// 创建对象使用new关键字
```

```js
基本算术表达式: + - * / %
一元算术运算符: + - ++ --  // 会把操作数转换为数字
位运算符: & | ~ << >> >>> // 操作数为32位整数, 最后一个运算符为无符号右移
比较运算符: < > <= >= == ===  // 注意区分最后两个
逻辑表达式: && || !
三目运算符: ? :  // 基本类似C++中的运算符

typeof  
(typeof value == "string") ? "'" + value + "'" : value;

delete  // 删除对象属性或者数组元素
var o = {x: 1, y: 2}
delete o.x;
"x" in o;  // false

in  // 希望左操作是一个字符串, 右侧对象拥有一个名为左操作数值的属性名则返回true
var point = { x: 1, y: 1};
"x" in point;  // true
"z" in point;  // false

instanceof  // 希望做操作数是一个对象, 如果左侧对象是右侧类的实例, 则返回true
var d = new Date();
d instanceof Date;  // true
d instanceof Number;  // false
```



#3. 语句
---

**定义变量**

```js
// var name_1 [= value_1] [ ,..., name_n [= value_n]] 
var x = 2,
    f = function(x) { return x * x; },
    y = f(x);
```

**定义函数**

```js
function funcname([arg1 [, arg2 [..., argn]]]) {
    statement;
}   // 函数定义格式
var f = function(x) {
    return x * x;
}
```

**if else语句**

```js
if (expression) {
    // 执行代码块1
} 
else if (expression) {
    // 执行代码块2
}
else {
    // 执行代码块3
}
```

**switch语句**

```js
// case表达式和expression的值按照===比较
switch(expression) {
    case value1:
    //执行代码1
    break;
    case value2:
    //执行代码2
    break;
    case value3:
    //执行代码3
    break;
    default:
    //执行代码4
    break;
}
```

**四种循环while,do-while,for,for-in**

```js
while(expression) {
    statement;
}
do {
    statement;
} while(expression);  // 注意分号结尾
for(initialize; test; increment) {
    statement;
}
for(variable in object) {  // 常用语遍历对象属性成员
    statement;
}
```

- break语句跳转到循环(最内层)或者其他语句结束
- continue语句终止本次循环并开始下一次循环
- return语句让解释器跳出函数体执行(`直接return; 返回的是undefined`)
- throw语句触发或者抛出一个异常


#4. 对象
---

> JavaScript对象都是关联数组
> 所有JavaScript对象都从Object.prototype继承属性

**创建对象**

三种方法:

```js
var point = {x: 0, y: 1};  // 对象直接量
var point1 = new Object();  // 使用new关键字创建一个空对象
var point2 = Object.create({x: 1, y: 2});  //使用Object.create()静态函数point3继承属性x和y
```

**查询和设置属性**

```js
var first = point.x;  // .语法
var second = point["y"];  // []语法
```

**删除属性**

```js
delete point.x;
(delete point["y"]) == true;  // delete删除成功或没有副作用则返回true
```

> delete不可删除继承属性, 只能删除自有属性

**检测属性**

```js
"x" in point;  // true
"toString" in point  // true, 继承属性
point.hasOwnProperty("toString");  // false, hasOwnProperty方法检测是否是自有属性
point.propertyIsEnumerable("toString");  // false, 检测自有属性且属性可枚举
```

**枚举属性**


```js
for(p in point) {
    if (!point.hasOwnProperty(p))   // 跳过非自有属性
        continue;
    else if (typeof point[p] === "function")  // 跳过方法
        continue;
    else:
        console.log(p);
}
```


**属性getter和setter**

```js
var point = {
    date_prop: value,
    // 存取器属性都是成对定义的函数
    get accessor_prop() {},
    set accessor_prop(value) {}
};
```

**对象的三属性**

- 原型prototype
- 类class
- 可扩展性extensible attribute

```js
var p = {x: 1};
var o = Object.create(p);
p.isPrototypeOf(o);  // true, o集成自p
Object.prototype.isPrototypeOf(p);  // true, p继承自Object.prototype

//返回任意对象的类
function classof(o) {
    if (o === null) {
        return "Null";
    }
    if (o === undefined) {
        return "Undefined";
    }
    return Object.prototype.toString.call(o).slice(8, -1);
}

```

**序列化对象**

```js
var point = {
    x: 1,
    y: {
        z: [false, null, ""]
    }
}
s = JSON.stringify(point);  // JSON字符串化对象
p = JSON.parse(s);  // 还原对象, p是point的深拷贝
```


#5. 数组
---

> JavaScript数组是动态的额, 数组元素无类型(可以存放任意类型), `数组是对象的特殊形式`

**创建数组**

```js
var misc = [1.1, true, "a",];  // 数组直接量允许结尾有逗号
var undes = [,,];  // 两个undefined的元素
var a = new Array();  //构造一个空数组
var a = new Array(10);  //创建指定长度数组, 数组中没有值(没有undefined)
var a = new Array(1, 2, 3, "test");  // 构造带元素的数组
```

**数组的读取和写入**

`下标访问, 从零开始`

> 通过`var a = new Array(10); `可以创建稀疏数组

```js
var a = [];
a.push(5);  // 末尾添加元素
a.pop();  // length长度减一,并删除元素
delete a[0];  // 删除元素, delete操作不影响数组的length值
```

**多维数组**

使用数组的数组进行近似, 风格和访问与C++中类似

**数组方法**

```
var a = [1, 2, 3];
a.join();  // 数组拼接, 默认使用,拼接成字符串, 是String.split的逆方法
a.reverse();  // 数组逆序
a.sort();  // 数组排序
a.concat([1, 2]);  // 数组拼接, 返回新数组
a.slice(0, 2);  // 数组切片
a.splice(删除起始位置[, 删除个数[, 插入元素]]);  // 在数组中插入或删除元素
a.push(), a.pop();  // 数组模拟栈
a.unshift(), a.shift();  // 数组头部添加和删除元素
a.toString();  // 每个元素字符串化

/* ECMAScript 5数组方法 */
forEach();  // 从头到尾遍历数组, 为每个元素调用指定函数
map();  // 对每个元素调用指定元素, 并返回数组
filter();  // 返回满足函数逻辑的元素的数组(子集)
every(), some(); // 对数组元素应用指定函数判定, 返回true或false
reduce(操作函数[, 可选传递给操作函数的初始参数])
indexOf(), lastIndexOf();  // 搜索整个数组具有给定值的元素
Array.isArray([]);  // true, 判断是否为数组
```

#6. 函数
---

> 函数即对象

 **函数定义**
 
```js
function funcname(参数) {  // 可以赋值给变量, 存在一个arguments指向实参对象的引用
    /* 执行代码 */
}
```

- 函数length属性代表函数参数的数量
- prototype属性
- call()和apply()方法
- bind()方法, 将一个函数绑定到对象中, 作为对象的方法
- Function()构造函数, 可以传入任意数量的字符串实参, 最后一个实参表示的文本就是函数体(`不能用来实现闭包`), 匿名

```js
f.call(o)  // 类似于o.f()
f.apply(o)

var f = new Function("x", "y", "return x * y;");
// 等价于var f = function(x, y){ return x * y; }
```

#7. 类和模块

> JavaScript中类的实现`基于原型`继承机制, JavaScript中的类是动态可继承的


**关键字new调用构造函数,自动创建一个新对象,构造函数的prototype属性被用作新对象的原型,构造函数对已创建的对象初始化**

```js
function Range(from, to) {
    this.from = from;
    this.to = to;
}
Range.prototype = {  // Range()构造函数自动使用Range.prototype作为新Range对象的原型
    constructor: Range,  // 显式设置构造函数反向引用
    includes: function(x) { return this.from <= x && x <= this.to; },
    foreach: function(f) {
        for(var x = Math.ceil(this.from); x <= this.to; x++) f(x);
    },
    toString: function() { return "(" + this.from + "..." + this.to + ")" }
};

var r = new Range(1, 3);
```

**检测任意对象的类:**

1. 使用instanceof运算符
2. constructor属性(前两种方法在多个执行上下文场景中无法正常工作)
3. 构造函数的名字

```js
var r = new Range(1, 3);
r.constructor === Range  // true

/* 除了没有名字的函数和不具有constructor对象都能够识别 */
function type(o) {
    var t, c, n;
    if (o === null) 
        return "null";
    if (o !== o)
        return "nan";
    if ((t = typeof o) !== "object") 
        return t;  // 识别原始值的类型和函数
    if ((c = classof(o)) !== "Object")
        return c;  // 识别出大多数内置类型
    if (o.constructor && typeof o.constructor === "function" &&
        (n = o.constructor.getName()))
        return n;  //如果对象构造函数名字存在, 则返回
    return "Object";  // 无法识别类型返回"Object"
}
function classof(o) {
    return Object.prototype.toString.call(o).slice(8, -1);
}
Function.prototype.getName = function() {
    if ("name" in this)
        return this.name;
    return this.name = this.toString().match(/function\s*([^(]*)\(/)[1];
}
```

**子类**

B是A的子类

```js
B.prototype = inherit(A.prototype);
B.prototype.constructor = B;
```
