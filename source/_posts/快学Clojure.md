title: 快学Clojure
date: 2016-03-19 00:13:12
tags: Clojure
---

> Why? 为了更好的了解并发和函数式语言.

## 简介

- `Clojure`是一套现代的Lisp语言的动态语言版(Lisp方言)
- 函数式语言, 编译型语言
- 最开始Clojure运行在JVM上, 所有有很好的扩平台特性.
- 函数式语言的`不变性`使它们很好的支持并发

<!--more-->

### 环境部署

1. 下载[IntelliJ IDEA](https://www.jetbrains.com/idea/)
2. Preference -> Plugins -> Browse Repositories -> 查找[Cursive](https://cursive-ide.com/)
3. 重启IDEA

使用[Leiningen](http://leiningen.org/)作为Clojure的命令行构建工具

```
# 安装
$ brew update
$ brew install leiningen

# 查看lein的命令
$ lein help

# 进行Clojure的REPL
$ lein repl

# 使用Clojure创建一个可运行的Clojure的项目
$ lein new app hello-world

# 进入创建的项目, 然后执行
$ lein run

# 在项目内对代码打包
$ lein uberjar

# 使用java运行jar包
$ java -jar target/uberjar/hello-world-0.1.0-SNAPSHOT-standalone.jar
```


## 语法

> 每学习一门新的编程语言, 我觉得都应该对应以前学习过的语法, 找出不同之处, Clojure的语法感觉还是挺个性的.

- `Clojure`代码由一个个form组成， 即写在小括号里的由空格分开的一组语句
- Clojure的注释以 `;`(分号)开始

```
; operator未操作符(函数或者加减乘除等), 后面均为操作数
(operator operand1 operand2 ... operandn)
```

### 变量

```
; 分号后均为注释, def定义一个变量
; 其中user=> 为lein repl给出的命令行提示符(下同)
user=> (def a 1)
#'user/a
user=> (println a)
1
```

### 基本类型和操作

- Clojure基本数据类型包括`Number, String, Map, Keyword, Vector, List, Set`, 均为`immutable`, 对基本数据类型的更改都会在生成一个新的拷贝
- 其中关键字`Keyword`类似于`Ruby`张的符号(symbol)或`Java`中的保留字符串


```
user=> (def a 1)  ; 整数, java.lang.Long
user=> (def s "hello")  ; 字符串, java.lang.String
:a  ; keyword
user=> (def v [1 2 3])  ; Vector数组, clojure.lang.PersistentVector
user=> (def a `(1 2 3))  ; List链表, clojure.lang.PersistentList
user=> #{1 2 3}  ; 使用#{}创建一个Set, 类型为clojure.lang.PersistentHashSet
; 通过Vector来创建Set
user=> (set [1 2 3])



# 创建一个hashMap, :a是clojure.lang.Keyword类型, 等下会讲到
user=> (def mymap  {:a 1 :b 2 :c 3})
#'user/mymap
user=> (class mymap)
clojure.lang.PersistentArrayMap
; map支持很多类型的key，但推荐使用keyword类型作为map的key
; keyword类型和字符串类似, 但做了一些优化。
(class :a) ; => clojure.lang.Keyword
```

 - 集合操作

```
; 判断是否是集合
user=> (coll? '(1 2 3))  ; => true
user=> (coll? [1 2 3])  ; => true
; 判断是否为List
user=> (seq? '(1 2 3))  ; => true
user=> (seq? [1 2 3])  ; => false
; cons用以向列表或向量的起始位置添加元素
user=> (cons 4 [1 2 3])  ; => (4 1 2 3)
user=> (cons 4 '(1 2 3))  ; => (4 1 2 3)
; 向集合中插入元素, 对于List, 数据会在起始位置插入, 而对于Vector, 则在末尾位置插入
user=> (conj [1 2 3] 4)
user=> (conj '(1 2 3) 4)
user=> (conj #{1 2 3} 4)
#{1 4 3 2}

; 从Map中查找元素
user=> (def mymap  {:a 1 :b 2 :c 3})
user=> (mymap :a)
1
user=> (mymap :d)  ;查找不到的元素会返回nil
nil
; dissoc溢出Map中的元素
user=> (dissoc mymap :a)
{:b 2, :c 3}
```

- 基本操作

```
(+ 1 1)  ; 加 
(- 2 1)  ; 减
(* 1 2)  ; 乘
(/ 2 1)  ; 除
(= 1 1)  ; 等号
(not true)  ; 逻辑非
(< 1 2)  ; < > = 
```

- if语句

```
(if condition-form
    then-form
    optional-else-form)
```

> `切记: Cloju使用nil和false表示逻辑假, 其他所有值的均为真`


```
user=> (if false "hello" "world")  ; 输出结果为world
```

### 函数

```
; 简单的函数调用
(+ 1 2 3)
```

- 使用`defn`来自定义一个函数
- 函数定义使用`(defn 函数名 函数注释 参数列表, 函数体)`
- 函数体可以由多个form构成, Clojure将最后一个form的执行结果作为返回值

```
# hello为函数名, ""中为函数注释, []为参数, 函数体可以包含多个form
user=> (defn hello "just a test method" [sentence] (println sentence))
#'user/hello
user=> (hello "andrew liu")
andrew liu
; 变参函数，即把&后面的参数全部放入一个序列
user=> (defn print0 [& args]
  #_=> (str "parameter is " args))
#'user/myprint
user=> (print0 "Andrew" "Liu")
"parameter is (\"Andrew\" \"Liu\")"
; 固定参数和可变参数的结合
user=> (defn print1 [name & args]
  #_=> (println name)
  #_=> (str args))
#'user/print1
user=> (print1 "Andrew" 1 2 3)
Andrew
"(1 2 3)"
```

- 函数重载

```
user=> (defn func-overload
  #_=> ([] (str "no arg"))
  #_=> ([name] (str "arg " name))
  #_=> ([name_a name_b] (str "arg " name_a " " name_b)))
#'user/func-overload
user=> (func-overload)
"no arg"
user=> (func-overload "andrew")
"arg andrew"
user=> (func-overload "andrew" "liu")
"arg andrew liu"
```

- 匿名函数

使用`fn`来创建一个匿名函数, 在Python中使用lamda来创建匿名函数.

```
# 定义一个匿名函数变量
user=> (def no_name (fn [name] (str "hello " name)))
#'user/no_name
user=> (no_name "andrew")
"hello andrew"
```

### 函数式

- map, reduce, partial

```
; 对给定序列的每一个元素做inc操作(映射)
user=> (map inc [0 1 2 3])
(1 2 3 4)

; reduce操作有三个参数, 第一个为双参数函数, 第二个为初始值, 第三个为集合,  
user=> (reduce (fn [x y] (+ x y)) 0 [1 2 3])
6

; partial绑定一个参数为固定值
user=> (map (partial * 2) [1 2 3 4])
(2 4 6 8)
```


### 宏

- 通过defmacro来定义macro


## 并发

- Future 可以让我们在另一个线程去执行任务，它是异步执行的，所以调用了future之后会立即返回
- Promise是一个承诺，我们定义了这个promise，就预期后续会得到相应的result



## 参考链接

- [Clojure repl Connection Closed](http://stackoverflow.com/questions/30799764/clojure-repl-connection-closed)
- [https://siddontang.gitbooks.io/lean-clojure/content/concurrency.html](https://siddontang.gitbooks.io/lean-clojure/content/concurrency.html)
- [clojure学习资源](http://chunyemen.org/archives/642)
- [Getting Started with Clojure](http://clojure-doc.org/articles/tutorials/getting_started.html)
- [clojure-doc-en2ch](https://code.google.com/archive/p/clojure-doc-en2ch/wikis/chapter1.wiki)
- [Clojure 学习笔记梳理](https://segmentfault.com/a/1190000003074935)
- [X分钟速成Y](https://learnxinyminutes.com/docs/zh-cn/clojure-cn/)
