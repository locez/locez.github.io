---
title: Protocol Buffers 食用方法
date: 2021-01-01 09:16:24
categories:
 - [linux]
 - [protobuf]
 - [protocol buffers]
 - [cpp]
tags:
 - linux
 - protobuf
 - protocol buffers
 - cpp
---

##### AUTHOR: [Locez](http://locez.com)
##### VERSION: 1


### 1 Protocol Buffers 是什么？
---

***Protocol Buffers*** 是 google 开发的一种开源的、跨平台的、可扩展的结构化数据序列化机制。使用 Protocol BUffers 可以定义只定义一次结构化的数据，然后通过生成的代码，在各种语言的数据流中都可以轻松高效的读取和写入定义好的结构化数据。

**注**: 因代码仓库名称为 **protobuf**，以下为了行文简单， 统一使用 **protobuf** 指代 **Protocol Buffers**


#### 1.1 Protobuf 的优势是什么？
---

Protobuf 是一种结构化的，序列化和反序列化的机制，那么对比最常用的 **json**、 **xml** 有什么优势呢？

 - 序列化与反序列化执行效率高
 - 序列化产物较小，在网络传输中可以更加节省带宽
 - 支持直接生成各种语言的代码，访问接口统一舒适

一个基本的 **proto** 文件中的结构体定义如下（proto2 语法）:

 ``` protobuf
 message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;
}
 ```

<!-- more -->

可以看到结构体中定义了字段的类型和名字，以及字段的对应的唯一的编号。其中编号用来定位在二进制数据中，字段的位置，所以一个结构体一旦使用了，就不应该再被更改。因为 protobuf 使用编号定位字段，所以序列化的时候不会带上 **key**，只会有 **value**，这就使得序列化产物非常小。通常 1 到 15 号用来标识一些比较频繁出现的数据。

相对应地，一个 **json** 字符串如下:

``` json
{
  "name": "Locez",
  "id": 1234,
  "email": "locez@locez.com"
}
```
上述的字符串就是 **json** 序列化后的产物，可以看到， json 序列化时会带上 **key**，使得序列化产物比 **protobuf** 大的多( **xml** 同理)，其次就是类似于 **json** 这种高度可读的数据，并没有保存数据的类型，在解析的时候，需要不断的判断或断言，找到正确的数据类型，然后绑定到这个数据类型上面去，而 **protobuf** 在序列化的时候会带入数据类型，从而提高它在反序列化时的效率。



<!-- more -->
### 2 安装 protobuf
---

要使用 protobuf，首先需要生成 protobuf 目标语言的源代码，protobuf 提供了一个名为 **protoc** 的二进制程序，用来生成目标语言源代码。

Gentoo
```
# emerge --ask protobuf
```

Ubuntu
```
# apt-get install libprotobuf-dev protobuf-compiler
```

Arch
```
# pacman -S protobuf
```


#### 2.1 protoc
---
使用 protoc 来生成目标语言的源代码， 主要使用的参数有两个:
  - **-I** 指定 *proto* 文件的搜索更目录，当在一个 **proto** 文件中使用相对路径 **import** 了另一个 **proto** 文件时，使用 **-I** 配置搜索的根目录
  - **--language_out=dir** 生成目标语言的代码，且存放到 **dir** 中
```
$ protoc --help
Usage: protoc [OPTION] PROTO_FILES
Parse PROTO_FILES and generate output based on the options given:
  -IPATH, --proto_path=PATH   Specify the directory in which to search for
                              imports.  May be specified multiple times;
                              directories will be searched in order.  If not
                              given, the current working directory is used.
                              If not found in any of the these directories,
                              the --descriptor_set_in descriptors will be
                              checked for required proto file.
...
...
  --cpp_out=OUT_DIR           Generate C++ header and source.
  --csharp_out=OUT_DIR        Generate C# source file.
  --java_out=OUT_DIR          Generate Java source file.
  --js_out=OUT_DIR            Generate JavaScript source.
  --objc_out=OUT_DIR          Generate Objective-C header and source.
  --php_out=OUT_DIR           Generate PHP source file.
  --python_out=OUT_DIR        Generate Python source file.
  --ruby_out=OUT_DIR          Generate Ruby source file.
```

### 3 protobuf 使用实战
---

接下来，将使用 `cpp` 进行一些 protobuf 使用的实战，演示 protobuf 的能力

项目结构主要如下：

``` bash
~/protobuf-examples
$ tree .
.
├── cpp
│   └── read_write.cpp
└── proto
    └── AddressBook.protoo
```

生成 **cpp** 代码的命令如下：

``` bash
$ protoc -I ./proto/ --cpp_out=cpp/proto proto/*
```

为了便于工程上的管理，本文的 example 最终采用 cmake 构建，项目结构如下， 生成 cpp 文件的操作在 **CMakeLists.txt** 中完成

``` bash
~/protobuf-examples
$ tree .
.
├── cpp
│   ├── CMakeLists.txt
│   ├── Makefile
│   ├── pb2json.cpp
│   ├── proto -> ../proto
│   └── read_write.cpp
└── proto
    ├── AddressBook.proto
```


#### 3.1 protobuf 读写、序列化与反序列化

该 example 的 proto 文件如下：

``` cpp
syntax = "proto2";

package address_book;

message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    optional string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }
  // 可重复的
  repeated PhoneNumber phone = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

Protobuf 所生成的 cpp 代码，以 **object.set_variable(value)** 的形式进行赋值操作，以 **object.variable()** 的形式进行访问

``` cpp
#include <iostream>

#include "pb/AddressBook.pb.h"

int main(int argc, char **argv)
{
    // 声明一个地址簿对象
    address_book::AddressBook address_book;
    // repeated 标识符的字段为 0 - n 个，接口是 add_xxx(), 获取一个用于读写的指针
    auto people = address_book.add_people();
    // 使用　set_xxx() 方法填写属性
    people->set_email("locez@locez.com");
    people->set_id(0);
    people->set_name("locez");
    auto phone_number = people->add_phone();
    phone_number->set_number("123456789");
    // enum 枚举通过命名空间访问
    phone_number->set_type(address_book::Person::PhoneType::Person_PhoneType_HOME);

    std::cout << "readable: \n"
              << address_book.DebugString() << std::endl;
    auto serialized_data = address_book.SerializeAsString();

    std::cout << "binary: ===\n"
              << serialized_data << "\n===\n"
              << " length: " << serialized_data.length() << std::endl << std::endl;

    address_book::AddressBook address_book_new;
    // 反序列化
    address_book_new.ParseFromString(serialized_data);
    std::cout << "readable: \n" << address_book_new.DebugString() << std::endl;

    // repeadted 标识符的字段为 0 - n 个，访问需要遍历
    for (int i = 0; i < address_book_new.people_size(); i++)
    {
        auto people_new = address_book_new.people(i);
        std::cout << "name: " << people_new.name();
    }

    return 0;
}
```
输出如下：

``` bash
./bin/read_write
readable:
people {
  name: "locez"
  id: 0
  email: "locez@locez.com"
  phone {
    number: "123456789"
    type: HOME
  }
}

binary: ===

)
locezlocez@locez.com"
        123456789
===
 length: 43

readable:
people {
  name: "locez"
  id: 0
  email: "locez@locez.com"
  phone {
    number: "123456789"
    type: HOME
  }
}

name: locez
```

#### 3.2 Protobuf 与 json 的互相转换

protobuf 提供了与 json 互相转换的能力，以便于更好的在各种系统之间共享数据

proto 文件如下：

``` protobuf
syntax = "proto3";

package json_person;

message Person {
  string name = 1;
  int32 id = 2;
  string email = 3;

  enum PhoneType { MOBILE = 0; HOME = 1; WORK = 2; }

  message PhoneNumber {
    string number = 1 [json_name = "phone_number"];
    PhoneType type = 2 [json_name = "phone_type"];
  }

  PhoneNumber phone = 4;
}
```

protobuf 提供了一些序列化选项和反序列化选项，用来影响与 json 转换的具体行为，具体的例子如下：

``` cpp
#include <iostream>

#include <google/protobuf/util/json_util.h>

#include "proto/JsonPerson.pb.h"

void print(const std::string &title, const std::string &option, const std::string &content)
{
    std::cout << title << "  (" << option << "):\n"
              << content << "\n\n";
}

int main(int argc, char **argv)
{
    json_person::Person person;
    person.set_name("locez");
    person.set_email("locez@locez.com");
    person.mutable_phone()->set_type(json_person::Person::PhoneType::Person_PhoneType_MOBILE);
    person.mutable_phone()->set_number("1000000");
    std::string result;
    // 使用默认选项序列化成 json， 会忽略默认的值，如果带有 json_name 则会按照 json_name 定义的名字序列化该字段
    google::protobuf::util::MessageToJsonString(person, &result);
    print("serailize1", "default option", result);

    google::protobuf::util::JsonOptions serial_options;
    // 在 field 中指定了 json_name 的选项，但是序列化时，保留原有的字段名字
    serial_options.preserve_proto_field_names = true;
    result = "";
    google::protobuf::util::MessageToJsonString(person, &result, serial_options);
    print("serialize2", "perserve proto field names = true", result);

    serial_options = google::protobuf::util::JsonOptions();
    // add whitespace 选项会使得序列化后的 json 更加美观可读
    serial_options.add_whitespace = true;
    result = "";
    google::protobuf::util::MessageToJsonString(person, &result, serial_options);
    print("serialize3", "add whitespace = true", result);

    serial_options = google::protobuf::util::JsonOptions();
    // 默认值的字段也序列化
    serial_options.always_print_primitive_fields = true;
    result = "";
    google::protobuf::util::MessageToJsonString(person, &result, serial_options);
    print("serialize4", "always_print_primitive_fields = true", result);

    serial_options = google::protobuf::util::JsonOptions();
    serial_options.always_print_primitive_fields = true;
    // 枚举序列化成对应的 int 型数字，而不是对应的字面量字符串
    serial_options.always_print_enums_as_ints = true;
    result = "";
    google::protobuf::util::MessageToJsonString(person, &result, serial_options);
    print("serialize5", "always_print_enums_as_ints = true", result);

    // 这个 json 字符串多了一个 unknown_field 的字段，该字段没有在 proto 文件里面定义
    std::string json_string = R"({"name":"locez","id":0,"email":"locez@locez.com","phone":{"phone_number":"1000000","phone_type":0,"unknown_field":0}})";
    json_person::Person p;
    auto status = google::protobuf::util::JsonStringToMessage(json_string, &p);
    if (status.ok())
    {
        print("deserialize1", "default option", p.DebugString());
    }
    else
    {
        print("deserialize1 failed", "default option", status.error_message().ToString());
    }

    google::protobuf::util::JsonParseOptions deserialize_options;
    // 忽略不存在的字段，只解析存在的可以解析的字段
    deserialize_options.ignore_unknown_fields = true;
    status = google::protobuf::util::JsonStringToMessage(json_string, &p, deserialize_options);
    if (status.ok())
    {
        print("deserialize2", "ignore_unknown_fields = true", p.DebugString());
    }
    else
    {
        print("deserialize1 failed", "ignore_unknown_fields = true", status.error_message().ToString());
    }
    return 0;
}
```

输出如下：

``` bash
./bin/pb2json
serailize1  (default option):
{"name":"locez","email":"locez@locez.com","phone":{"phone_number":"1000000"}}

serialize2  (perserve proto field names = true):
{"name":"locez","email":"locez@locez.com","phone":{"number":"1000000"}}

serialize3  (add whitespace = true):
{
 "name": "locez",
 "email": "locez@locez.com",
 "phone": {
  "phone_number": "1000000"
 }
}


serialize4  (always_print_primitive_fields = true):
{"name":"locez","id":0,"email":"locez@locez.com","phone":{"phone_number":"1000000","phone_type":"MOBILE"}}

serialize5  (always_print_enums_as_ints = true):
{"name":"locez","id":0,"email":"locez@locez.com","phone":{"phone_number":"1000000","phone_type":0}}

deserialize1 failed  (default option):
(phone) unknown_field: Cannot find field.

deserialize2  (ignore_unknown_fields = true):
name: "locez"
email: "locez@locez.com"
phone {
  number: "1000000"
}
```


### 总结
---
未完待续

### 参考资料
  - 本文代码仓库-[protobuf_examples](https://github.com/locez/protobuf_examples)，文中的所有示例代码都可以在此找到