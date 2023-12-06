---
title: 其他细节
description: 其他细节
---


## Proto 版本

区分三个版本

| 名称                               | 说明                   | 例子          | 备注                                                              |
|----------------------------------|----------------------|-----------------|------------|
| protoc                      | 编译器                   | `3.21.7`| [protobuf](https://github.com/protocolbuffers/protobuf) 不同语言可能基于版本不一样   |
| grpc                      | grpc                   | `1.50.0`| [grpc](https://github.com/grpc/grpc) protobuf上RPC扩展   |
| protobuf-gradle-plugin                      | gradle插件                   | `0.8.19`| [protobuf-gradle-plugin](https://github.com/google/protobuf-gradle-plugin) 帮助gradle: 1：集成`protoc`命令， 2：source目录布局 |

所以关系是：

1. protoc 是底座
2. grpc 是扩展
3. protobuf-gradle-plugin 只是个辅助工具

这样松散的关系会导致： 版本的不兼容， protoc vs grpc vs protobuf-gradle-plugin， 目前 Apihug 测试通过的版本集合是

| 名称                               | 版本     |子包     | ApiHug Version |
|----------------------------------|----------------------|----------------------|-----------------|------------|
| protoc                      | `3.21.7`| `protobuf-java` & `protobuf-java-util` |`[0.3.5-RELEASE,)`|
| protobuf-gradle-plugin      | `0.8.19`| `com.google.protobuf:protobuf-gradle-plugin`|`[0.3.5-RELEASE,)`|
| grpc                        | `1.50.0`| `grpc-stub` `grpc-protobuf`, `grpc-core` |`[0.3.5-RELEASE,)`|

版本控制的细节，在ApiHug的 gradle 插件， 和 BOM 版本控制中已经预设好，兼容问题已经得到充分测试;

除非项目中自己强制指定版本，可能会导致兼容问题。⚠️

## Version Catalog

TBD

## Refer

1. [protobuf-gradle-plugin - git](https://github.com/google/protobuf-gradle-plugin)
2. [protobuf - git](https://github.com/protocolbuffers/protobuf)
3. [grpc - git](https://github.com/grpc/grpc)
