title: Template小记
date: 2015-09-20 13:59:52
tags: C++
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.


## 基本

**类模板**需要注意在类声明外部进行成员函数模板的实现, 需要使用`完整的类型限定符`

- 模板参数可以定义为默认值(类似与函数的默认值)
- 模板参数不仅可以是类型, 还可以是普通值(`非类型模板参数`)
- 类内使用类型模板参数中的某个类型时, 需要使用typename. (`typename T::iterator pos;`)
- 类中成员函数可以使用新的模板
- 模板的声明和实现都包含在头文件中(`包含模式`)

<!--more-->

```c
// MyStack.h
#ifndef _STACK_H_
#define _STACK_H_

#include <iostream>
#include <stdexcept>
#include <vector>

template <typename T>
class Stack {
public:
    void push(const T&);
    void pop();
    T top() const;
    bool empty() const {
        return elems.empty() ? true : false;
    }
private:
    std::vector<T> elems;
};
#endif

// MyStack.cpp
#include "MyStack.h"

// 注意是MyStack<T>
template <typename T>
void Stack<T>::push(const T& elem) {
    elems.push_back(elem);
}

template <typename T>
void Stack<T>::pop() {
    if (!elems.empty()) {
        std::cout << "out of range!" << std::endl;
    }
    elems.pop_back();
}

template <typename T>
T Stack<T>::top() const {
    if(elems.empty()) {
        std::cout << "out of range!" << std::endl;
    }
    return elems.back();
}
```

```c
// 非类型模板参数, 实例化时 NotTypeStack<int, 10> num_stack;
template <typename T, int MAX_SIZE>
class NotTypeStack {
public:
    NotTypeStack();
    void push(const T&);
    void pop();
    T top() const;
    bool empty() const {
        return cur_size == 0;
    }
    bool full() const {
        return cur_size == MAX_SIZE;
    }
private:
    T elems[MAX_SIZE];
    int cur_size;
};
```

## 模板特化

通过模板的特化, 对优化基于某个特定类型的实现, `特化时要特化该类模板中的所有成员函数`

```
// 特化使用<>
template <>
class Stack<std::string> {  //针对string特化
public:
    void push(const std::string&);
    void pop();
    std::string top() const;
    bool empty() const {
        return elems.empty() ? true : false;
    }
private:
    std::vector<std::string> elems;
};
```

