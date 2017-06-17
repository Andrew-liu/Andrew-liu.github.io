title: MatLab入门手册
date: 2015-04-05 09:50:22
tags: Paper
---


本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


#1. MatLab简介和安装
---

> - MatLab是综合高性能的数值计算软件, 集成了数值计算和可视化, 提供大量内置函数, 广泛用于科学计算等领域.
> - Simulink是基于MatLab的框图设计环境, 用来对各种动态系统进行建模, 分析和方针.
> - 学习目的: 为了完成当前作业和以为科研工作的需求


<!--more-->

安装包请在[MatLab官网](http://cn.mathworks.com/products/matlab/)进行下载和安装.


MatLab窗口>>

1. 命令窗口
2. 当前目录
3. 工作区


#2. MatLab基础
---

Matlab如果没有定义变量名, 系统将计算结果暂存在ans临时变量中.
优先级: 表达式从左向右运算, 指数运算优先级最高, 乘除次之(`help precedence`查看优先级)

**常用操作命令**

- clc 清空敏玲窗口
- diary 日志文件命名
- who 列出工作空间的变量

```
save my_var.mat  % 保存工作区变量
load my_var.mat  % 加载文件中变量到工作区
```


**运算符号**

```
+ - * / \ ^ '  % 加 减 矩阵乘法 右除 左除 次方 矩阵共轭转置
.* ./ \. .^ .'  % 数组乘法 数组右除 数组左除 数组次方 矩阵转置

>> a = round(rand(3) * 10)
a =
     8     9     3
     9     6     5
     1     1    10
>> b = magic(3)  % 定义矩阵a和矩阵b
b =
     8     1     6
     3     5     7
     4     9     2
>> c1 = a * b  % 矩阵叉乘
c1 =
   103    80   117
   110    84   106
    51    96    33
>> c2 = a .* b  % 矩阵点乘, 矩阵对应元素位置的元素相乘
c2 =
    64     9    18
    27    30    35
     4     9    20
>> e =a^2  % 矩阵的次方, 表示a * 2
e =
   148   129    99
   131   122   107
    27    25   108
>> d = a.^2  % 矩阵的乘方, 矩阵中每个元素二次方
d =
    64    81     9
    81    36    25
     1     1   100
>> format short, pi  % format用于更改显示的输出格式
ans =
    3.1416
>> format long, pi
ans =
   3.141592653589793
>> iskeyword  % 查询关键字
ans = 
    'break'
    'case'
    'catch'
    'classdef'
    'continue'
    'else'
    'elseif'
    'end'
    'for'
    'function'
    'global'
    'if'
    'otherwise'
    'parfor'
    'persistent'
    'return'
    'spmd'
    'switch'
    'try'
    'while'
>> class(a)  % 获取定义的数据类型
ans =
double
>> a = cast(a, 'uint8')  % 改变数据类型
a =
    8    9    3
    9    6    5
    1    1   10
>> class(a)
ans =
uint8
```

带`.`的运算方式表示对矩阵元素的运算, 矩阵的右除是一般意义的除法, `a./b = b.\a`

> MatLab默认的输出格式为双精度(`double`)


**变量**

- 变量名区分大小写, 以字母开头, 后跟字母数字下划线


##数组

> 数组作为Matlab存储和运算的基本单元

**数组创建**

```
>> a = [1 2 3]  % 直接创建, 以空格或者逗号隔开
a =
     1     2     3
>> a = 0 : 1 : 3  % x = a:inc:b a和b为起始数和终止数, inc为间隔步长
a =
     0     1     2     3
>> a = linspace(1, 3, 3)  % 等间距线性创建法, a = linspace(a, b, n), 在a和b区间取n个点
a =
     1     2     3
>> a = logspace(1, 3, 3)  % 等间距对数创建法
a =
          10         100        1000
```

**数组访问**

```
a =
          10         100        1000
>> a(2)  % 索引访问, 从1开始
ans =
   100
>> a(2 : 3)  % 按块访问
ans =
         100        1000
>> a(2 : end)
ans =
         100        1000
```


**多维数组**

- 直接创建(一位数组的创建方式), 同行元素用空格和逗号隔开, 不同行用分号隔开
- 在`新建变量`的窗口, 更改变量名, 插入数据
- 大规模数据可以使用`导入数据`导入工作空间
- 使用已有函数

**常用标准数组**

- eye生成单位矩阵
- ones生成全1数组
- rand生成随机数组, 服从均匀分布
- randn生成随机数组, 服从正态分布
- zeros生成全0数组
- diag生成对角矩阵

```
>> a = -3:3
a =
    -3    -2    -1     0     1     2     3
>> k = find(a > 0)  % 找到符合条件的下标
k =
     5     6     7
>> sort(a)  % 对数组排序
ans =
    -3    -2    -1     0     1     2     3
```

##字符串

字符串是用`单引号`括起来的一系列字符的组合

```
>> str = 'hello world'  % 定义字符串
str =
hello world
>> size(str)  % 查看字符串的大小
ans =
     1    11
>> u = mat2str(pi * eye(2))  % 将矩阵转化为字符串
u =
[3.14159265358979 0;0 3.14159265358979]
>> class(u)
ans =
char
>> disp(u)  % 打印字符串
[3.14159265358979 0;0 3.14159265358979]
>> size(u)
ans =
     1    39
>> t = 23  
>> tempText = [ 'Temperature is ', num2str(t), 'C']  # 字符串的拼接, 使用num2str将数字转换为字符串
tempText =
Temperature is 23C
```

##关系运算符

关系运算符主要比较两个同维数的数组的大小


#3. 矩阵运算
---

常用函数列举

```
chol(A)  % 对矩阵A进行Cholesky分解
det(A)  % 矩阵A的行列式
eig(A)  % 矩阵A的特征值和特征向量
inv(A)  % 矩阵A的逆
svd(A)  % 矩阵A的奇异值
eye(r, c)  % 生成r * c的单位矩阵
magic(n)  % 生成n*n的魔幻矩阵
ones(r, c)  % 生成r*c的全1矩阵
rand(r, c)  % 生成r*c的元素值0和1之间的随机矩阵
zeros(r, c)  % 生成r*c的全0矩阵
cond(A)  %利用奇异值分解求矩阵Ａ的范数
```
**求行列式**

```
>> for i = 1:3
A = magic(i + 2)
a(i) = det(A)  % 矩阵行列式
disp('矩阵:')  % 打印字符串 
disp(A)
disp('矩阵的行列式')
disp(a(i))
end
```

稀疏矩阵

```
>> a = [0 0 0 5; 0 1 0 0; 1 5 0 0; 0 0 0 3]
a =
     0     0     0     5
     0     1     0     0
     1     5     0     0
     0     0     0     3
>> as= sparse(a)  % 创建稀疏矩阵
as =
   (3,1)        1
   (2,2)        1
   (3,2)        5
   (1,4)        5
   (4,4)        3
>> af = full(as)  % 还原矩阵
af =
     0     0     0     5
     0     1     0     0
     1     5     0     0
     0     0     0     3
```


##线性方程组

**恰定方程组**是方程组个数和未知数个数相同的方程组, 使用`左除`求解.


#4. MatLab编程基础

Matlab可以像C一样编程, 编写执行命令的脚本和函数功能的模块, 文件以`.m`为后缀

打开M文件编辑器:

1. 新建->脚本
2. 在命令行输入edit命令, 或者`edit filename`命令


```
% 一个简单的脚本文件
echo on % 脚本文件内容显示在命令窗口
t = 0 : pi/20 : 2 *pi;
num = input('输入数字:');  % 提示用户输入内容
disp(num);  % 显示结果
echo off  %关闭命令行显示
```

##流程控制


**for循环**

```
for x = array  % x为循环变量, array是条件数组
    commands  % 循环执行的代码
end

% example
for i = 1: 1: 10
    a(i) = sin(i * pi /5)
end
a  % 输入a
```

**while循环**

```
while expression
    commands
end

% example
num = 10;
while num >1
    num = num / 2
end
```

**if条件结构**

```
if expression 
    commands
elseif 
    commands
else
    commands
end
```

**switch分支选择结构**

```
switch expression
    case test_expression1
        commands1
    case test_expression2
        commands2
    otherwise
        commands3
end

% example
x = input('输入需要换算的长度数值cm:');
unit = input('选择转换单位 (1 in, 2 ft, 3 m. 4 mm, 5 cm):');
switch unit
    case {'inch', 'in', 1}
        y = x * 2.54;
    case {'feel', 'ft', 2}
        y = x * 2.54 / 12;
    case {'meter', 'm', 3}
        y = x / 100;
    case {'centermeter', 'cm', 4}
        y = x;
    case {'milimeter', 'mm', 5}
        y = x * 10;
    otherwise
        disp('Unkonwn Units');
        y = NaN;
end
disp(y)
```

**try-catch结构**

try用于捕获try后语句的异常, 交给catch语句处理异常
```
try 
    commands
catch 
    commands
end

% example
x = rand(4 ,2)
y = magic(3)
try
    z = x * y
catch 
    z = NaN
    disp('两矩阵维数不同, 计算错误!')
end
disp(lasterr)
disp(lasterror)
```

**continue,break,return关键字的应用场景与其他语言基本相同**


##4.6. M函数文件

M函数文件与M脚本文件的不同:

- M函数文件第一行必须是function引导的声明语句, 成为函数声明行
- 函数执行中, 函数体内变量临时建立工作区, 称为函数工作区
- M函数文件可以调用M脚本文件
- M函数文件中可以创建一个或多个函数

绘制函数$y=e^{x/3}sinex(x)$在区间$[0, 4π]$的曲线

```
function y = sinex(x)
% sinex.m
y = exp(-x / 3).*sin(3 * x)


% 命令行调用函数
>> fh  = @sinex
>> ezplot(fh, [0, 4 * pi, -1, 1]);
```


#6. 帮助
---

Matlab所有函数都有详细的帮助文档, 通过一下的方式可以更好的使用文档:

- 命令行输入`doc functionname`(完整的文档)
- 输出`functionname(`程序进进行智能文档提示(速度慢)
- 命令行输入`help mean`(简单的文档)



#5. 参考链接
---

- [官方学习手册](http://cn.mathworks.com/help/matlab/learn_matlab/matrices-and-arrays.html)
- `<MatLab从入门到精通, 周建兴>`
- [官方文档](http://cn.mathworks.com/help/matlab/index.html)

