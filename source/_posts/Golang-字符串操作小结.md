title: Golang 字符串操作小结
date: 2016-04-29 08:01:58
tags: Go
---


本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


> 本文形式为, 给出一个函数接口, 然后匹配相关example. 关注函数主要集中在`strings`和`strconv`

<!--more-->

## 字符串比较

**函数接口**

```
// Compare比较字符串的速度比字符串内建的比较要快
func Compare(a, b string) int
```

```
    fmt.Println(strings.Compare(string("hello"), string("haha")))  // 1
    fmt.Println(strings.Compare(string("hello"), string("world"))) // -1
    fmt.Println(strings.Compare(string("hello"), string("helloworld")))  // -1
    fmt.Println(strings.Compare(string("hello"), string("hello")))  //0
```

## 字符串查找

**字符串查找的相关接口**

```
// 判断给定字符串s中是否包含子串substr, 找到返回true, 找不到返回false
func Contains(s, substr string) bool

// 在字符串s中查找sep所在的位置, 返回位置值, 找不到返回-1
func Index(s, sep string) int

// 统计给定子串sep的出现次数, sep为空时, 返回1 + 字符串的长度
func Count(s, sep string) int
```

```
    fmt.Println(strings.Contains("seafood", "foo"))  // true
    fmt.Println(strings.Contains("seafood", "bar"))  // false
    fmt.Println(strings.Contains("seafood", ""))  // true
    fmt.Println(strings.Contains("", ""))  // true

    fmt.Println(strings.Index("chicken", "ken"))  // 4
    fmt.Println(strings.Index("chicken", "dmr"))  // -1
    
    
    fmt.Println(strings.Count("cheeseeee", "ee"))  // 3
    fmt.Println(strings.Count("five", ""))  // 5
```

## 字符串重复

```
// 重复s字符串count次, 最后返回新生成的重复的字符串
func Repeat(s string, count int) string
```

```
fmt.Println("ba" + strings.Repeat("na", 2))  // banana
```

## 字符串替换

```
// 在s字符串中, 把old字符串替换为new字符串，n表示替换的次数，小于0表示全部替换
func Replace(s, old, new string, n int) string
```

```
    fmt.Println(strings.Replace("oink oink oink", "k", "ky", 2))  // oinky oinky oink
    fmt.Println(strings.Replace("oink oink oink", "oink", "moo", -1))  // moo moo moo
```

## 字符串删除

```
// 删除在s字符串的头部和尾部中由cutset指定的字符, 并返回删除后的字符串
func Trim(s string, cutset string) string

// 删除首尾的空格
```

```
// 删除首部和尾部的 ! 和空格
fmt.Printf("%q\n", strings.Trim(" !!! Achtung! Achtung! !!! ", "! "))  // "Achtung! Achtung"

fmt.Printf("%q\n", strings.TrimSpace(" \t\n a lone gopher \n\t\r\n"))  // "a lone gopher"
```

## 字符串大小写转换

```
// 给定字符串转换为英文标题的首字母大写的格式(不能正确处理unicode标点)
func Title(s string) string

// 所有字母转换为小写
func ToLower(s string) string

// 所有字母转换为大写
func ToUpper(s string) string
```


```
fmt.Println(strings.Title("her royal highness"))  // Her Royal Highness

fmt.Println(strings.ToLower("Gopher123"))  // gopher123

fmt.Println(strings.ToUpper("Gopher"))  // GOPHER
```

## 字符串前缀后缀


> 前缀和后缀的判断均为大小写敏感

```
// 判断字符串是否包含前缀prefix
func HasPrefix(s, prefix string) bool

// 判断字符串是否包含后缀suffix, 
func HasSuffix(s, suffix string) bool
```

```
    fmt.Println(strings.HasPrefix("Gopher", "Go"))  // true
    fmt.Println(strings.HasPrefix("Gopher", "go"))  // false
    fmt.Println(strings.HasPrefix("Gopher", "C"))  // false
    fmt.Println(strings.HasPrefix("Gopher", ""))  // true
    
    fmt.Println(strings.HasSuffix("Amigo", "go"))  // true
    fmt.Println(strings.HasSuffix("Amigo", "O"))  // false
    fmt.Println(strings.HasSuffix("Amigo", "Ami"))  // false
    fmt.Println(strings.HasSuffix("Amigo", ""))  // true
```

## 字符串分割

**函数接口**

```
// 把字符串按照sep进行分割, 返回slice(类似于python中的split)
func Split(s, sep string) []string

// 去除字符串s中的空格符, 并按照空格(可以是一个或者多个空格)分割字符串, 返回slice
func Fields(s string) []string

// 当字符串中字符c满足函数f(c)时, 就进行字符串s的分割
func FieldsFunc(s string, f func(rune) bool) []string
```

**使用example**

```
func main() {
    fmt.Printf("%q\n", strings.Split("a,b,c", ","))  // ["a" "b" "c"]
    fmt.Printf("%q\n", strings.Split("a man a plan a canal panama", "a "))  // ["" "man " "plan " "canal panama"]
    fmt.Printf("%q\n", strings.Split(" xyz ", ""))  // [" " "x" "y" "z" " "]
    fmt.Printf("%q\n", strings.Split("", "Bernardo O'Higgins"))  // [""]
    fmt.Printf("%q\n", strings.Split("1 og.txt", " "))
}  // ["1" "og.txt"]

    fmt.Printf("Fields are: %q\n", strings.Fields("  foo bar  baz   "))  // Fields are: ["foo" "bar" "baz"]
    
    f := func(c rune) bool {
        return !unicode.IsLetter(c) && !unicode.IsNumber(c)
    }
    // 表示按照非字母, 非数字来分割字符串
    fmt.Printf("Fields are: %q\n", strings.FieldsFunc("  foo1;bar2,baz3...", f))  // Fields are: ["foo1" "bar2" "baz3"]
```

## 字符串拼接

**三种拼接方案:**

- 直接用`+=`操作符, 直接将多个字符串拼接. 最直观的方法, 不过当数据量非常大时用这种拼接访求是非常低效的.
- `用字符串切片([]string)装载所有要拼接的字符串，最后使用strings.Join()`函数一次性将所有字符串拼接起来。在数据量非常大时，这种方法的效率也还可以的。
- 利用Buffer(`Buffer是一个实现了读写方法的可变大小的字节缓冲`)，将所有的字符串都写入到一个Buffer变量中，最后再统一输出.


**函数接口**

```
// bytes.Buffer的方法, 将给定字符串追加(append)到Buffer
func (b *Buffer) WriteString(s string) (n int, err error)

// 字符串拼接, 把slice通过给定的sep连接成一个字符串
func Join(a []string, sep string) string
```

**三种字符串拼接方式的性能测试(出自参考链接的文章)**

```
package main

import (
    "bytes"
    "fmt"
    "strings"
    "time"
)

func main() {
    var buffer bytes.Buffer

    s := time.Now()
    for i := 0; i < 100000; i++ {
        buffer.WriteString("test is here\n")
    }
    buffer.String()  // 拼接结果
    e := time.Now()
    fmt.Println("taked time is ", e.Sub(s).Seconds())

    s = time.Now()
    str := ""
    for i := 0; i < 100000; i++ {
        str += "test is here\n"
    }
    e = time.Now()
    fmt.Println("taked time is ", e.Sub(s).Seconds())

    s = time.Now()
    var sl []string
    for i := 0; i < 100000; i++ {
        sl = append(sl, "test is here\n")
    }
    strings.Join(sl, "")
    e = time.Now()
    fmt.Println("taked time is", e.Sub(s).Seconds())
}
```

**运行结果如下**

```
taked time is  0.004145088
taked time is  13.78821647
taked time is 0.024005696
```

## 字符串转换

字符串转化的函数在`strconv`中

- `Append*`函数表示将给定的类型(如`bool, int`等)转换为字符串后, 添加在现有的字节数组中`[]byte`
- `Format*`函数将给定的类型变量转换为string返回
- `Parse*`函数将字符串转换为其他类型

```
// 字符串转整数
func Atoi(s string) (i int, err error)
// 整数值换字符串
func Itoa(i int) string
```

```
str := make([]byte, 0, 100)
str = strconv.AppendInt(str, 123, 10)  // 10用来表示进制, 此处为10进制
str = strconv.AppendBool(str, false)
str = strconv.AppendQuote(str, "andrew")
str = strconv.AppendQuoteRune(str, '刘')
fmt.Println(string(str))  // 123false"andrew"'刘'

s := strconv.FormatBool(true)
fmt.Printf("%T, %v\n", s, s)  // string, true
v := 3.1415926535
s32 := strconv.FormatFloat(v, 'E', -1, 32)
fmt.Printf("%T, %v\n", s32, s32)  // string, 3.1415927E+00
s10 := strconv.FormatInt(-44, 10)
fmt.Printf("%T, %v\n", s10, s10)  // string, -44


num := strconv.Itoa(1234)
fmt.Printf("%T, %v\n", s, s)  // int, 1023
fmt.Printf("%T, %v\n", num, num)  // string, 1234
```



## 参考链接

- [go语言中高效字符串拼接](http://www.birdcat.cn/golang%E5%AD%A6%E4%B9%A0/go%E8%AF%AD%E8%A8%80%E4%B8%AD%E9%AB%98%E6%95%88%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%8B%BC%E6%8E%A5.html#6815723-twi-1-96951-c0a587a04d612ecbd4c03c162fa4c28d)
- [7.6 字符串处理](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh%2F07.6.md)

