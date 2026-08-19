---
layout : post
date: 2014-10-01 22:18:23 +0800
title: 偶遇JavaScript
tags: JavaScript
comments: true
---

`这两周基本在被挖掘机作业折磨着, 还好最后交上了, 这么快就国庆了, 南大的适应阶段结束了`
`虽然掉了钱包, 但是我感觉这或许是一个重新开始的好事, 一切都不错, 生活也逐渐走上了正轨`
`加油吧, 朝着自己的目标走, 而不是为了拿高分而学习, 以自身兴趣为基础, 爱生活, 爱Code, 爱老婆, 国庆快乐....`

#1. What can we use JavaScript for?

- make websites respond to user interaction 
- build apps and games (e.g. blackjack)
- access information on the Internet (e.g. find out the top trending words on Twitter by topic)
- organize and present data (e.g. automate spreadsheet work; data visualization)

<!--more-->


#2. 常用语法
1. confirm()显示弹窗
2. prompt()显示带输入交互的弹窗
3. console.log() 输出
4. "some word".substring(x, y) where x is where you start chopping and y is where you finish chopping the original string,子串
5. var name = data; 定义变量


#3. function
定义函数的格式格式:

```
var sayHello = function(name)
{
    console.log('Hello ' + name);
}; //最后大括号外面有个分号

```

```
Math.random(), that variable will equal a number between 0 and 1.
Math.floor(Math.random() * 5 + 1);运行原理:
1. Math.random()产生0到1之间的随机数,
2. 乘5后生成0到5之间随机数,
3. Math.floor()取整数位,
4. 最后加1使0到4变成1到5
```


```
.toUpperCase() or .toLowerCase()  将输入全部转为大写或者全部转为小写
```

#4. Array的属性
` .length`计算array的长度或者说元素个数
1. `Heterogeneous array`异构数组 :异构数组可以包含不同的数据类型(包括object)
2. 一个数组中可以包含另一个数组,也就是二位数组
3. ` jagged array`锯齿状数组 : 二维数组每行数据个数不同

#5. object对象
> object可以看作一个键值对(其中键不可以是数字)
`storing all relevant information in one place—an object.`

object包含多个属性(目录标签) 属性的冒号后跟属性的值(value);


##5.1. 创建object的方法
1. object literal notation. object的文字符号
2. object constructor.object构造函数


```
//object literal notation 
var myObj = {
    type: 'fancy',
    disposition: 'sunny'
};
var emptyObj = {};

//object constructor
var myObj = new Object();

//增加键的两种方法
myObj["name"] = "Charlie";
myObj.name = "Charlie";
```

##5.2. 访问属性的value的方法
1. ` dot notation`.点号. `ObjectName.PropertyName (e.g., bob.name)`
2.  `bracket notation`[]括号 .`ObjectName["PropertyName"]`,注意需要使用双引号


##5.3. method is like a function
1. 在object定义一个方法的格式`objectName. propertyName = function(){ to do something };`
2. 调用方法 :`ObjectName.methodName()`
3. 方法可以用来改变对象的属性
4. 方法可以基于属性做出一些计算

###5.3.1. this as a placeholder
this.property 始终表示调用方法的object, 类似C++中的this

###5.3.2. define a Constructor
> 构造函数是使用new关键字创建一个object的方式(最基础的是Object构造函数)

格式 : function FunctionName(para){something;}



```
function Person(name,age) {
  this.name = name;
  this.age = age;
  this.species = "Homo Sapiens"; //object中定义一个默认属性
}
//使用自定义的构造函数
var bob = new Person("Bob Smith", 30);
```

- 构造函数定义方法的格式


        function someObject() {
            this.propertyName = propertyValue;
            this.someMethod = function() {};
        }
        var someObj = {
        aProperty: value,
        someMethod: function(some, params) { }
        };

```
//构造函数中定义方法
function Rectangle(height, width) {
  this.height = height;
  this.width = width;
  this.calcArea = function() {  //对象方法
      return this.height * this.width;
  };
  // put our perimeter function here!
  this.calcPerimeter = function()
  {
      return (this.height + this.width) * 2;
  }
}
//调用其中的方法
var rex = new Rectangle(7,3);
var area = rex.calcArea();

//Array object, 生成一个数组对象?
var family = new Array();

//object可以作为参数
var ageDifference = function(person1, person2) {
    return person1.age - person2.age;
}
```

###5.3.3. What Are Objects For?
- 对象提供了一种方式去描述整个世界
- 属性属于对象, 类似于变量

未完待续, 其中一些内容需要进一步整理, 目前还在学习阶段...
