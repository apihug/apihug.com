---
title: Protobuf 简介
description: Proto Buffer 如何使用, Proto Buffer 手册，Proto Buffer 是什么，Protobuf好处，Protobuf不足，Protobuf是什么？
---

## Protobuf

Protobuf即Protocol Buffers，是Google公司开发的一种跨语言和平台的序列化数据结构的方式，是一个灵活的、高效的用于序列化数据的协议。与XML和JSON格式相比，Protobuf更小、更快、更便捷。

Protobuf是跨语言的，并且自带编译器(protoc)，只需要用protoc进行编译，就可以编译成Java、Python、C++、C#、Go等多种语言代码，并可以直接使用。

定义了要处理的数据的数据结构之后，就可以利用ProtoBuf的代码生成工具生成相关的代码。只需使用 Protobuf 对数据结构进行一次描述，即可利用各种不同语言(proto3支持C++, Java, Python, Go, Ruby, Objective-C, C#)或从各种不同流中对你的结构化数据轻松读写。


### 工作原理


通过 `.proto` 文件中定义 `protocol buffer message` 类型，来指定你想如何对序列化信息进行结构化。

每一个 protocol buffer message 是一个信息的小逻辑记录，包含了一系列的 name-value 对

这是一个最基本的message例子 

```proto

syntax = "proto3";

option java_multiple_files = true;
option java_package = "com.test.google";

message Model {
int32 id = 1;
int32 shopId = 2;
optional string name = 3;
optional int32 categoryId = 4;
optional DecimalValue markPrice = 5;
optional bool onShelf = 6;
repeated ModelParameterValue parameter = 7;
}

message DecimalValue {
uint32 scale = 1;
uint32 precision = 2;
bytes value = 3;
}

message ModelParameterValue {
optional int32 id = 1;
optional int32 parameterId = 2;
optional string parameterName = 3;
optional string parameterValue =4;
optional int32 modelId = 5;
}

```

1. 第一行指明当前使用的是`proto3`语法, 如果不指定则默认为`proto2`.
1. `java_multiple_files` 指的是是否生成多个`java`文件， `false`则只编译`ModelOuterClass`文件
1. `java_package` 指定生成的java文件的所在包
1. `optional` 表示该字段为可选字段
1. `repeated` 表示该字段可以复用，常用于集合定义 


### 其他对象

除上面的一些数据结构外， 还有一些复杂的数据结构:

### Enum

```proto

enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
}
```

注意 ⚠️ 3.0 后， element_number: 必须从0开始, 0表示类型的默认值, 32-bit integer; 

是否会导致你不想用的结果？ 看具体业务需求， 当然protoc 会再最后自动加一个： `UNRECOGNIZED(-1)`

所以几种做法： 

1. 第一个也就是 `0` 永远用 `NA` 标志， 也就是不用
2. 业务逻辑层需要处理  `NA  + UNRECOGNIZED` 两个非预期的数值！


### Wrapper

3.0 以后， 不可以设置default值， 这个给处理带来了巨坑， 因为程序逻辑中一般有 空值额外处理逻辑！

于是乎引入了 wrapper 去处理这个错误导致的问题:

```proto
import "google/protobuf/wrappers.proto";

message Tester {
    google.protobuf.BoolValue email = 1016;
}

message BoolValue {
  // The bool value.
  bool value = 1;
}

```

#### Map

`repeated` 表示该字段可以复用，常用于集合定义, `map` 对应我们一般意义上的 `Map`;

建议里面放，同构的数据， 而不是 `Any`， 或者下面的 `Value` :

```proto
message Tester {
    map<string, google.protobuf.Value> extensions = 15;
}
```

⚠️ 根据官方 [Maps Features](https://protobuf.dev/programming-guides/proto3/#maps-features) 文档，
在 **hope** 框架里更建议使用兼容变形的 map 方式来定义 map 类型结构， 在语言实现上也能够兼容:

```proto
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

#### Any 

⚠️ Any & Value ⚠️ 在**hope** 元信息定义类不被采纳！

当无法明确定义数据类型的时候， 可以使用Any表示:

```proto

import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}

// `Any` contains an arbitrary serialized protocol buffer message along with a
// URL that describes the type of the serialized message.
//
// Protobuf library provides support to pack/unpack Any values in the form
// of utility functions or additional generated methods of the Any type.
//     {
//       "@type": "type.googleapis.com/google.protobuf.Duration",
//       "value": "1.212s"
//     }

message Any {

  string type_url = 1;

  // Must be a valid serialized protocol buffer of the above specified type.
  bytes value = 2;
}

```

这个是最不建议的！

#### Value

`Value` 貌似比 `Any` 好点点， 

```proto

message Tester {
    map<string, google.protobuf.Value> extensions = 9;
}

// `Value` represents a dynamically typed value which can be either
// null, a number, a string, a boolean, a recursive struct value, or a
// list of values. A producer of value is expected to set one of these
// variants. Absence of any variant indicates an error.
//
// The JSON representation for `Value` is JSON value.
message Value {
  // The kind of value.
  oneof kind {
    // Represents a null value.
    NullValue null_value = 1;
    // Represents a double value.
    double number_value = 2;
    // Represents a string value.
    string string_value = 3;
    // Represents a boolean value.
    bool bool_value = 4;
    // Represents a structured value.
    Struct struct_value = 5;
    // Represents a repeated `Value`.
    ListValue list_value = 6;
  }
}

```

## 参考

1. [protobuf](https://protobuf.dev/)
2. [protobuf Git](https://github.com/protocolbuffers/protobuf)
3. [protobuf gradle plugin](https://github.com/google/protobuf-gradle-plugin)
4. [protobuf intellj plugin](https://plugins.jetbrains.com/plugin/14004-protocol-buffers)
5. [API的设计 - 知乎 - 玩家翁伟​​](https://zhuanlan.zhihu.com/p/43809461) 一些思路， 这里设计完看到的，比较接近
6. [深入浅出：如何正确使用 protobuf](https://zhuanlan.zhihu.com/p/406832315)
7. [WSDL -WSDL is an XML notation for describing a web service](https://www.w3.org/TR/wsdl/)
8. [Thrift IDL](https://thrift.apache.org/docs/idl)
