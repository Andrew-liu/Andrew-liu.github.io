title: MySQL Small Cookbook
date: 2015-05-24 15:20:21
tags: [Database, SQL]
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.



> MySQL是一种关系型数据库(`RDBMS`), 数据库可以理解为相关文件的集合. 数据库和控制器数据库的软件称为数据库管理系统(`DBMS`)


> 数据库提供处理数据的方法: `SQL`

#基本概念

- 每个表由多个`行`和`列`组成
- 每行包含一个单独实体的数据, 称为`记录`
- 每一列包含与该记录相关的`一项数据`, 称为`属性`

<!--more-->

##安装
> 本博文中所有的SQL语句遵循`小写书写风格`, 不喜勿喷


```
$ brew install mysql
$ mysql -u root mysql


#设置开机启动
$ ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents 
#加载mysql
$ launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
#启动mysql服务器
$ mysql.server start
#关闭mysql服务器
$ mysql.server stop

#使用根用户
$ mysql -u root 
#创建用户密码
mysql> set password=password('123456');

#创建数据库 
mysql> create database firstdb;

#添加用户和密码, 并赋予firstdb的完全访问权限, 账户名为amdin, 密码为123456
mysql> grant all on firstdb.* to admin@localhost identified by '123456';  

#退出
mysql> exit
```

##初试数据库

```
#使用非根用户登陆数据库, 并使用firstdb
mysql> mysql -u admin -p123456 firstdb
```


###创建表

```
#创建
mysql> create table sales_rep(
    -> employee_number int,
    -> surname varchar(40),
    -> first_name varchar(30),
    -> commission tinyint
    -> );
    
#显示已有表
mysql> show tables;

#describe来检查表结构
mysql> describe sales_rep;
```

sales_rep为表名, employee_number, surname, first_name, commission为属性, int表示整型, varchar表示变长字符, tinyint表示小整数

###删除表和数据库

```
#创建一个表
mysql> create table com(id int);
#删除表使用drop关键字
mysql> drop table com;

#切换root用户, 创建数据库com
mysql> create database com;
#删除数据库
mysql> drop database com;
```


###插入/删除/修改记录

```
#插入数据 insert into 表名(要插入的属性名) values(属性值), 字符串使用单引号
mysql> insert into sales_rep values(1, 'Rive', 'Sol', 10);
mysql> insert into sales_rep values(2, 'Gordimer', 'Charlens', 15);
mysql> insert into sales_rep values(3, 'Serote', 'Mike', 10);

#一行添加数据
mysql> insert into sales_rep values
     >(1, 'Rive', 'Sol', 10),
     >(2, 'Gordimer', 'Charlens', 15),
     >(3, 'Serote', 'Mike', 10);

#从文件中加载数据, 格式load data local infile "文件名" into table 表名;
mysql> load data local infile "sales_rep.sql" into table sales_rep;
```

**删除记录**


```
# 删除employee_number为5的记录
mysql> delete from sales_rep where employee_number = 5;
```

**修改记录**

```
#修改employee_number的记录的commission为12
mysql> update sales_rep set commission = 12 where employee_number = 1;
```


###数据检索

```
#检索所有插入的数据
mysql> select * from sales_rep;
#检索surname为'Gordimer'的记录
mysql> select * from sales_rep where surname='Gordimer';
```

###模式匹配

`like和%`进行模糊查找

```
# 输出已surname已R开头的数据
mysql> select * from sales_rep where surname like 'R%';
```

###排序

**order by**

```
#数据按照surname排序输出, 当surname相同时, 使用first_name排序
mysql> select * from sales_rep order by surname, first_name;

#递减排序使用关键字desc, 默认使用升序asc
mysql> select * from sales_rep order by surname desc;
```

**多列排序**时, 使用逗号隔开排序规则, `order by排序优先次序为从左到右`

```
mysql> select ename, job, sal from emp  order by deptno asc, sal desc;
```

**按照字符串部分子串排序**

```
#按照job中最后两个字符进行排序
mysql> select ename, job from emp order by substr(job, length(job) - 1);
```


书中说: `找到字符串末尾(字符串长度)并减2, 则其实诶之就是字符串中倒数第二个字符`

> 但在我测试过程用应该是建1, 则是对最后两个字符排序(疑问?)

**根据数据项的键排序**

使用case语句


```
如果job是salesman, 按照comm, 否则, 按照sal排序
mysql> select ename, sal, job, comm from emp
    -> order by case when job = 'salesman' then comm else sal end;
```



###限制数据数量

limit

```
#按surname降序输出两行
mysql> select * from sales_rep order by surname desc limit 2;
```

从表中随机返回n条记录

- `order by`可以接受函数的返回值, 并使用它来改变结果集的顺序


```
select ename, job from emp order by rand() limit 5;
```


###最大值/最小值/计数/平均/综合


```
#查询commission的最大值
mysql> select max(commission) from sales_rep;
#查询最小值
mysql> select min(commission) from sales_rep;
#表中不重复surname的个数
mysql> select count(distinct surname) from sales_rep;
#commission的平均值
mysql> select avg(commission) from sales_rep;
#commission的总和
mysql> select sum(commission) from sales_rep;
#right()从字符串右端算起, 按位返回字符串
mysql> select right(date_joined, 5), right(birthday, 5) from sales_rep;
```

###去重

```
#使用distinct, 去掉查询字段相同的记录
```

###改变表结构

**添加列**

```
#给表添加一列名为data_joined, 类型为date
mysql> alter table sales_rep add date_joined date;
#添加一类名为year_born, 类型为year
alter table sales_rep add year_born year;
```

**修改列定义**

```
将year_born改为 列名为birthday, 类型为data
mysql> alter table sales_rep change year_born birthday date;
```

**重命名表**

```
mysql> alter table sales_rep rename cash_flow;
#恢复原来表名
mysql> alter table cash_flow rename to sales_rep;
```

**删除列**

```
#删除列名为enhancement_value的一列
mysql> alter table sales_rep drop enhancement_value;
```

###日期函数

```
#给date类型设置日期
mysql> update sales_rep set date_joined = '2014-02-15', birthday = '2000-02-14' where employee_number = 1;


#使用日期函数, 设置显示日期格式
mysql> select date_format(date_joined, '%m/%d/%y') from sales_rep;
# 使用year()输出年, month()输出月, dayofmonth()输出一个月的第几日
mysql> select year(birthday), month(birthday), dayofmonth(birthday) from sales_rep;
```

###高级查找(别名, concat, 多表查询, case表达式)

**as起别名**(类似pytho中import包时用as起别名)

```
mysql> select year(birthday) as year, month(birthday) as month, dayofmonth(birthday) as day from sales_rep;
```

**在别名的时候用别名做限定条件**

> from语句是在where之前完成的

```
#将查询结果作为内敛视图
mysql> select * from (select sal as salary, comm as commission from emp) x where salary < 5000;
```


**concat连接列**

将多列作为一列进行输出

```
#将first_name, 一个空格, surname连接在一起输出
mysql> select concat(first_name, ' ', surname) as name, month(birthday) as month from sales_rep order by month;
```


```
mysql> select concat(ename, ' works as a ', job) as msg from emp where deptno = 10;
```

###多表查询

创建两个表并插入数据

```
mysql> create table client(
    -> id int,
    -> first_name varchar(40),
    -> surname varchar(30)
    -> );
mysql> create table sales(
    -> code int,
    -> sales_rep int,
    -> customer int,
    -> value int
    -> );
    
mysql> insert into customer values
    -> (1, 'Yvaonne', 'Clegg'),
    -> (2, 'Johnny', 'Chaka'),
    -> (3, 'Winston', 'Powers'),
    -> (4, 'Patricia', 'Mankunku');
mysql> insert into sales values
    -> (1, 1, 1, 2000),
    -> (2, 4, 3, 250),
    -> (3, 2, 3, 500),
    -> (4, 1, 4, 450),
    -> (5, 3, 1, 3800),
    -> (6, 1, 2, 500);
```


```
code为1, 且两表中employee_number和sales_rep的记录输出, select后面部分列出要返回的字段
mysql> select sales_rep, customer, value, first_name, surname from sales, sales_rep where code = 1  and sales_rep.employee_number= sales.sales_rep;
```

**case表达式**

对select中的列值执行`if-else`操作

```
mysql> select ename, sal,
    -> case when sal <= 2000 then 'underpaid'
    -> when sal >= 4000 then 'overpaid'
    -> else 'ok'        #else语句是可选的
    -> end as status    #对case语句返回的列取别名
    -> from emp;
```



###查询中分组(不懂)

group by指的是按照某个属性分组, 与其他组互不干扰

```
#查询每个sales_rep的value值的和
mysql> select sales_rep, sum(value) as sum from sales group by sales_rep;
```


##常用类型

**数字类型**

- int(整型), 表示整数
- float/double分别表示单精度和双精度浮点数

**字符类型**

- char(M) 固定长度为M的字符串, 字符串长度不够会补上空格 , `搜索时大小写无关`
- varchar(M), 可变长字符串(`相比char一般比较节省内存`), `搜索时大小写无关`
- text, 最大65535个字符, `搜索时大小写无关`
- blob, 最大65535个字符, `搜索时大小写相关`

**日期和时间类型**

- date, 默认格式`YYYY-MM-DD`, 可以使用`date_format()`函数更改输出方式
- timestamp(M), 时间戳, `YYYYMMDDHHMMSS`, 可以指定不同长度的时间戳(`M只影响显示`)
- time, 格式`HH:MM:SS`

##表类型

|表类型|优点|缺点|
|---|---|---|
|静态表|速度快, 易缓存|要求更多的磁盘空间|
|动态表|占磁盘空间小|需要维护, 不易出问题后重建|
|压缩表|只读表类型, 占用很少磁盘空间|每条记录分开压缩, 不能同时访问|
|merge表|表尺寸小, 某些情况下速度快|eq_ref搜索慢, replace不能工作|
|heap表|散列索引, 最快|数据存在内存, 出现问题易丢失|


#高级SQL

三表连接查找

```
mysql> select sales_rep.first_name, sales_rep.surname,
    -> value, customer.first_name, customer.surname
    -> from sales, sales_rep, customer where sales_rep.employee_number = sales.sales_rep
    -> and customer.id = sales.customer;
```

sales_rep表的employee_number与sales的sales_rep关联
customer表的id和sales的customer相关, `构成了连接条件`

- `等值`连接是一种内连接, 通过两个表或者多个表相等条件, 将两个表的行组合到一个表中.(`内连接是连接的原始类型, 返回的每一行都包含来自每个表的数据`)

```
mysql> select ename, loc from emp, dept where emp.deptno = dept.deptno and emp.deptno = 10;

```
##内连接


```
#下面两句是等价的
mysql> select first_name, surname, value from customer, sales where customer.id = sales.customer;
#注意inner join 表名 on 相关字段的SQL语句
mysql> select first_name, surname, value from customer inner join sales on customer.id = sales.customer;
```


##左连接

左连接就是返回左边匹配行, 不考虑右边的表是否有相应的行. `返回匹配的全部行的必须是左表, left join关键字之前`

```
#语法
select field1, field2 from table1 left join table2 on field1 = field2;
```

```
mysql> select first_name, surname, value from sales left join customer on id = customer;
```

> 右连接连接的顺序与左连接相反

##union链接

union用来把不同的select结果连接成一个, 每个语句必须有相同个数的列


> union可能会进行去重处理, 不去重可以使用union all进行连接

> union语句中`order by`是在整个union上进行的, 如果只想在某一个上使用可以请用小括号, union默认不返回重复记录

```
#创建一个表用于union查询
mysql> create table old_customer(
    -> id int,
    -> first_name varchar(30),
    -> surnamr varchar(40));
#插入数据
mysql> insert into old_customer values
    -> (5432, 'Thulani', 'Salis'),
    -> (2342, 'Shahiem', 'Papo');
#两个查询语句有相同的列
mysql> select id, first_name, surname from old_customer union select id, first_name, surname from customer;
```

##查询结果添加到另一个表

```
#创建一个用于插入的表
mysql> create table customer_sales_values(
    -> first_name varchar(30),
    -> surname varchar(40),
    -> value int);
mysql> insert into customer_sales_values(first_name, surname, value)
    -> select first_name, surname, sum(value) from sales natural join customer group by first_name, surname;
```

#索引和查询优化

> 在MySQL中, 有四种类型的索引, `主键, 唯一索引, 全文索引和普通索引`

**主键(Primary Key)就是值唯一并且没有值为NULL的域的索引**

- 主键不能包含`NULL`

```
#创建表, 并设置主键
mysql> create table pk_test(
    -> f1 int not NULL,
    -> primary key(f1));
    
#已存在的表上创建主键, 语法: alter table tablename add primary key(fieldname1 [, fieldname2])
mysql> alter table customer modify id int not null, add primary key(id);
```

**普通索引容许重复的值**

```
#创建普通索引语法 mysql> create table(fieldname columntype, fieldname2 columntype, index[indexname](fieldname1 [,fieldname2...]));
#创建后添加索引: alter table tablename add index[indexname](fieldname1 [,fieldname2...]);
mysql> alter table sales add index(value);
```


**全文索引**

```
#创建全文索引语法 mysql> create table(fieldname columntype, fieldname2 columntype, fulltext[indexname](fieldname1 [,fieldname2...]));
mysql> insert into ft2 values
    -> ('Waiting for the barbarians'),
    -> ('In the Heart of the Country'),
    -> ('The master of Patersbury'),
    -> ('Writing and Being'),
    -> ('Heart of the Beast');
    -> ('Heart of the Beest'),
    -> ('The beginning and the End'),
    -> ('Master Master'),
    -> ('A barBarian at my door');
Query OK, 4 rows affected (0.00 sec)
#创建表后再添加全文索引:alter table tablename add fulltext[indexname](fieldname1 [,fieldname2...]);
mysql> create table ft(f1 varchar(255), f2 text, f3 blob, f4 int);
mysql> alter table ft add fulltext(f1, f2);
```
全文索引的用法

> 文本域查找与大小写无关

```
# match()匹配属性, against()匹配值
mysql> select * from ft2 where match(f1) against ('master');
```


**唯一索引除了不容许有重复的记录外, 与普通索引一样**

```
#创建普通索引语法 mysql> create table(fieldname columntype, fieldname2 columntype, unique(fieldname1 [,fieldname2...]));
mysql> create table ui_test(f1 int, f2 int, unique(f1));
mysql> insert into ui_test values(1, 2);
#f1域不能包含重复值
mysql> insert into ui_test values(1, 3);
ERROR 1062 (23000): Duplicate entry '1' for key 'f1'

#创建后添加索引: alter table tablename add unique[indexname](fieldname1 [,fieldname2...]);

```

##删除或者改变索引


```
#删除主键
alter table tablename drop primary key;
#删除普通, 唯一, 全文索引, 需要制定索引名
alter table tablename frop index indexname;
#显示全部索引名
show keys from tablename;
```

##选择索引

- 有查找需要使用索引的时候, 考虑创建索引
- 创建索引返回的行越少越好
- 私用短索引
- 不要创建太多的索引

#SQL Cookbook


##查找空值

查找空值切记不能用`=`操作, 会返回`Empty Set`

```
mysql> select * from emp where comm is null;
```

##空值转换为实际值返回

> 改变输入的形式, 但并不改变表中的数据

coalesce()函数有1个或者多个参数, comm非空时返回comm, null时返回0,  也可以用`case语句判断实现`

```
mysql> select coalesce(comm, 0) from emp;
```

##从一个表中查找与其他表不匹配的记录

`外连接`

```
mysql> select d.* from dept d left outer join emp e
    -> on (d.deptno = e.deptno) where e.deptno is null;
```


##插入更新删除

##插入技巧

##从一个表向另外的表中复制行

解决方案: 在insert语句后面紧跟一个用来产生索要插入行的查询


```
mysql> create table dept_east(
    -> deptno int,
    -> dname  varchar(30),
    -> loc varchar(30));
mysql> insert into dept_east(deptno, dname, loc)
    -> select deptno, dname, loc from dept
    -> where loc in('new york', 'boston');
```

##赋值表定义

> 只复制已有表的定义, 不复制其中的记录, 创建表的时候, 使用`一个不返回任何行的子查询, where的条件时钟为false`

```
mysql> create table dept_2
    -> as
    -> select * from dept where 1 = 0;
```

##修改技巧

```
mysql> update emp set sal = sal * 1.10 where deptno = 20;
```

##删除技巧

###删除所有记录

```
mysql> delete from dept_2;
```




#参考链接


- `<MySQL从入门到精通>(前四章)`
- `<SQL Cookbook>(前四章)`
[MySQL学习笔记4：完整性约束](http://www.cnblogs.com/nerxious/archive/2012/12/28/2837513.html)
[MySQL入门书籍和方法分享](http://cenalulu.github.io/mysql/mysql-book-for-newbie/)

