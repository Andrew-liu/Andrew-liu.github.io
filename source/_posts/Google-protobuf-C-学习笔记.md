title: Google protobuf(C++) 学习笔记
date: 2016-06-05 18:26:17
tags: C++
---


本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

>  `Google Protocol Buffers`简称 Protobuf, 是`Google`公司内部的混合语言数据标准. 它提供一种轻量, 高效的结构化数据存储结构. 

## 简介

**为什么学习protobuf?**

想通过protobuf的序列化来做一个C++的微型RPC框架.
本文主要是学习protobuf的使用, 大量参考官方文档.

<!--more-->

**为什么要使用protobuf?**

1. 因为`protobuf`是谷歌出的, 性能不过, 重要是的我是`谷歌脑残粉`(逃
2. 官方文档中提到一些protobuf的优点, protobuf灵活高效的结构化数据存储格式. 方便用于序列化, 适合做RPC的数据交换.
3. 相比`XML`, `protobuf`比 XML 更小、更快、更简单. 仅需要写一个`*.proto`文件描述需要的数据结构, protobuf会帮助你实现相关类和方法(自动化多好!).
4. 目前提供`C++, Java, Python, Go, C#等多种语言`的API


## 安装

**神奇Mac版homebrew帮你解决一切问题**

```
# 安装最新的protobuf
$ brew install protobuf

# 验证protobuf是否安装成功
$ protoc --version
# 在我电脑上的输出
libprotoc 3.0.0
```


## 学习笔记

**通过一个简单的官方例子来学习protobuf**

我们首先定义一个`addressbook.proto`, 通过`message`来定义需要序列化的数据结构, message内包含许多key-value对

```
# addressbook.proto
package tutorial;

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phone = 4;
}

message AddressBook {
  repeated Person person = 1;
}
```

**ps:**

1. `message`中有多种类型, `bool, int32, float, double, ,string and enum`以及内嵌的`message`
2. `message`内可以定义optional、required、repeated字段


**定义好*.proto文件后, 我们通过protobuf提供的protoc命令行工具来自动生成相关数据结构源码**

```
// 编译.proto文件
$ protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto

// 范例, 源目录为当前文件, 输出路径为当前文件
$ protoc -I=./ --cpp_out=./ ./addressbook.proto
```

- `-I`指明源文件所在目录
- `--cpp_out`指向想要输出的文件路径
- 最后一个参数为`.proto`所在的路径


## 使用protobuf


生成我们自定义的类后, 我们开始尝试使用生成的文件. `add_person.cpp`用于从用户的输入中, 创建一个Person并初始化一个Person对象, 并将该对象通过二进制的形式写入给定文件中.

```cpp
// 代码出自官方文档
// add_person.cpp

//
// Created by Andrew_liu on 16/4/17.
//
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// 基于用户输出填充Person message
void PromptForAddress(tutorial::Person* person) {
    cout << "Enter person ID number: ";
    int id;
    cin >> id;
    person->set_id(id);
    cin.ignore(256, '\n');

    cout << "Enter name: ";
    getline(cin, *person->mutable_name());

    cout << "Enter email address (blank for none): ";
    string email;
    getline(cin, email);
    if(!email.empty()) {
        person->set_email(email);
    }

    while(true) {
        cout << "Enter a phone number: ";
        string number;
        getline(cin, number);
        if(number.empty()) {
            break;
        }
        tutorial::Person::PhoneNumber* phone_number = person->add_phone();
        phone_number->set_number(number);

        cout << "Is this a mobile, home, or work phone(type: mobile/home/work): ";
        string type;
        getline(cin, type);
        if (type == "mobile") {
            phone_number->set_type(tutorial::Person::MOBILE);
        } else if (type == "home") {
            phone_number->set_type(tutorial::Person::HOME);
        } else if (type == "work") {
            phone_number->set_type(tutorial::Person::WORK);
        } else {
            cout << "Unknown phone type. Using default." << endl;
        }
    }
}

int main(int argc, char* argv[]) {
    GOOGLE_PROTOBUF_VERIFY_VERSION;
    if (argc != 2) {
        cerr << "Usage: " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
        return -1;
    }
    tutorial::AddressBook address_book;

    {
        fstream input(argv[1], ios::in | ios::binary);
        if (!input) {
            cout << argv[1] << ": File not found. Creating a new file. " << endl;
        } else if (!address_book.ParseFromIstream(&input)) {
            cerr << "Failed to parse address book." << endl;
            return -1;
        }
    }

    // add an address
    PromptForAddress(address_book.add_person());

    {
        // write new address book back to disk
        fstream output(argv[1], ios::out | ios::trunc | ios::binary);
        if(!address_book.SerializeToOstream(&output)) {
            cerr << "Failed to write address book." << endl;
            return -1;
        }
    }

    // delete all global object allocated by libprotobuf
    google::protobuf::ShutdownProtobufLibrary();
    return 0;
}
```

`list_person.cpp`通过给定的文件名读取Person对象, 并将每个Person对象输出.


```cpp
// list_person.cpp
// Created by Andrew_liu on 16/4/17.
//
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

void ListPeople(const tutorial::AddressBook& address_book) {
    for (int i = 0;  i < address_book.person_size(); i++) {
        const tutorial::Person& person = address_book.person(i);

        cout << "Person ID: " << person.id() << endl;
        cout << "Name: " << person.name() << endl;
        if (person.has_email()) {
            cout << "E-mail address: " << person.email() << endl;
        }

        for (int j = 0; j < person.phone_size(); ++j) {
            const tutorial::Person::PhoneNumber& phone_number = person.phone(j);

            switch (phone_number.type()) {
                case tutorial::Person::MOBILE:
                    cout << "Mobile phone #: ";
                    break;
                case tutorial::Person::HOME:
                    cout << "Home phone #: ";
                    break;
                case tutorial::Person::WORK:
                    cout << "Work phone #: ";
            }
            cout << phone_number.number() << endl;
        }
    }
}

int main(int argc, char* argv[]) {
    GOOGLE_PROTOBUF_VERIFY_VERSION;

    if (argc != 2) {
        cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
        return -1;
    }

    tutorial::AddressBook address_book;

    {
        fstream input(argv[1], ios::in | ios::binary);
        if (!address_book.ParseFromIstream(&input)) {
            cerr << "Failed to parse address book." << endl;
            return -1;
        }
    }

    ListPeople(address_book);
    google::protobuf::ShutdownProtobufLibrary();

    return 0;
}
```

完成后, 我们分别编译两个源文件, 编译命令如下:

```
# 编译add_person.cpp, 生成writer可执行文件
$ g++ -o writer add_person.cpp addressbook.pb.cc `pkg-config --cflags --libs protobuf`

# 编译list_person.cpp, 生成reader可执行文件
g++ -o reader list_person.cpp addressbook.pb.cc `pkg-config --cflags --libs protobuf`
```

图片为执行`writer`和`reader`的结果

![](http://ww3.sinaimg.cn/large/ab508d3djw1f2zvi7x4hxj21kw0ef45h.jpg)


## 实现RPC

```c
package news;
message NewsRequest {
    required string message = 1;
};
message NewsResponse {
    required string response = 1;
};
service NewsService {
    rpc News(NewsRequest) returns (NewsResponse);
};
```

## 参考链接

- [C++ Installation and Compile](https://github.com/google/protobuf/tree/master/src)
- [Protocol Buffer Basics: C++](https://developers.google.com/protocol-buffers/docs/cpptutorial#why-use-protocol-buffers)
