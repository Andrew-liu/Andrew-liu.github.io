---
title: Google Protobuf源码剖析(一)
date: 2016-11-07 21:52:00
tags: C++
desc: protobuf c++ google
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

> 很久之前写过一篇[Google protobuf(C++) 学习笔记](http://andrewliu.in/2016/06/05/Google-protobuf-C-%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/). `google protobuf`被大量用于公司的RPC通信中作为序列化和序列化工具, 高于JSON和XML的性能值得拥有. 刚好最近有时间, 准备强读一发`google protobuf源码`

## 前提

本文所有所有示例均基于官方示例`addressbook.proto`:

<!--more-->

```c
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

- `required`字段初值是必须要提供的, 否则字段的便是未初始化的, 序列化的时候必须对`required`初始化
- `optional`字段如果未进行初始化，那么一个默认值将赋予该字段
- `repeated`字段可以理解为`数组`, 

> 每个变量后的数字为标签, 用于标示了字段在二进制流中存放的位置

运行`protoc -I=./ --cpp_out=./ ./addressbook.proto`通过`protoc`来生成 `.h和.cpp`文件.

那么`.h文件`中到底生成了什么呢?


```cpp
# Person类的基类为::google::protobuf::Message类
class Person : public ::google::protobuf::Message

# enum类型的数据, 可以通过Person::MOBILE来访问
typedef Person_PhoneType PhoneType;
static const PhoneType MOBILE =
    Person_PhoneType_MOBILE;
static const PhoneType HOME =
    Person_PhoneType_HOME;
static const PhoneType WORK =
    Person_PhoneType_WORK;
    
# required和optional的普通类型, 产生的函数族都是差不多的. 对于每个字段会生成一个has函数、clear清除函数、set函数、get函数
// required string name = 1;
bool has_name() const;  # has_xxx
void clear_name();  # clear_xxx
const ::std::string& name() const;
void set_name(const ::std::string& value);
void set_name(const char* value);
void set_name(const char* value, size_t size);
::std::string* mutable_name();
::std::string* release_name();
void set_allocated_name(::std::string* name);

# repeated类型有些不同, 没有has_xxx函数族
// repeated .tutorial.Person.PhoneNumber phone = 4;
int phone_size() const;
void clear_phone();
const ::tutorial::Person_PhoneNumber& phone(int index) const;
::tutorial::Person_PhoneNumber* mutable_phone(int index);
::tutorial::Person_PhoneNumber* add_phone();
::google::protobuf::RepeatedPtrField<::tutorial::Person_PhoneNumber >*
      mutable_phone();
const ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >&
      phone() const;
```


## Protobuf主要类


- `Message`类, 是一个抽象层, 记录了一个proto文件里的所有内容. `Message`继承自`MessageLite`
- `MessageLite`类, 是一个轻量级的接口协议, 这个接口由所有协议的消息对象来实现, 这个类中包含大量定义的虚函数和纯虚函数. 使用`MessageLite`来生成代码, 需要在.proto中加入下面这行

```
option optimize_for = LITE_RUNTIME;
```

- `Arena`类主要用于协议消息的内存分配和释放, 并且分配内存是线程安全的.

```cpp
// 内存块的链表组织结构
  // Blocks are variable length malloc-ed objects.  The following structure
  // describes the common header for all blocks.
  struct Block {
    void* owner;   // &ThreadCache of thread that owns this block, or
                   // &this->owner if not yet owned by a thread.
    Block* next;   // Next block in arena (may have different owner)
    // ((char*) &block) + pos is next available byte. It is always
    // aligned at a multiple of 8 bytes.
    size_t pos;
    size_t size;  // total size of the block.
    GOOGLE_ATTRIBUTE_ALWAYS_INLINE size_t avail() const { return size - pos; }
    // data follows
  };

// Arena核心的初始化函数
void Arena::Init() {
  lifecycle_id_ = lifecycle_id_generator_.GetNext();
  blocks_ = 0;
  hint_ = 0;
  owns_first_block_ = true;
  cleanup_list_ = 0;
  // options为构造函数的参数, 一个配置结构体ArenaOptions
  if (options_.initial_block != NULL && options_.initial_block_size > 0) {
    GOOGLE_CHECK_GE(options_.initial_block_size, sizeof(Block))
        << ": Initial block size too small for header.";

    // Add first unowned block to list.
    Block* first_block = reinterpret_cast<Block*>(options_.initial_block);
    first_block->size = options_.initial_block_size;
    first_block->pos = kHeaderSize;
    first_block->next = NULL;
    // Thread which calls Init() owns the first block. This allows the
    // single-threaded case to allocate on the first block without taking any
    // locks.
    first_block->owner = &thread_cache();
    SetThreadCacheBlock(first_block);
    AddBlockInternal(first_block);
    owns_first_block_ = false;
  }
```

- `Reflection`类, 是一个用于动态访问和修改协议消息各种变量域的类(也就是我们常说的反射机制), 该类只在Message中实现(`MessageLite中没有`)
- `Descriptor`类, 用于描述协议消息,通过`Message::GetDescriptor()`来获取Message对应的


待续...
