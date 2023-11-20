---
title: Protobuf DSL 实现 OAS
description: 使用 Proto Buffer 实现DSL, DSL 描述 OAS OpenAPI Specification
---

## Mix

### 定义

1. [Protobuf](./002_protobuf_basic.md) 即Protocol Buffers，是Google公司开发的一种跨语言和平台的序列化数据结构的方式，包含GRPC 
2. [The OpenAPI Specification](https://github.com/OAI/OpenAPI-Specification) 定义了一个标准、跟编程语言无关的 HTTP API接口描述 (defines a standard, programming language-agnostic interface description for HTTP APIs);  
3. 领域特定语言(domain-specific language)（DSL）,是一种旨在特定领域下的上下文的语言。这里的领域是指某种商业上的（例如银行业、保险业等）上下文，也可以指某种应用程序的（例如 Web 应用、数据库等）上下文。

### 目的

借助 Protobuf 实现 DSL 用以定义 OpenAPI； WHY?

1. 更易用： Protobuf 语法极简，长时间业界实践， OAS, 通用标准，简单易懂。
2. 更结构化：严格于 json/yml 纯格式， 静态语法校验， 更结构化，更严谨，完善包分发体系。
3. 更平滑：protoc 和各宿主语言之间的桥梁， 让设计和实现之间无缝平移。

## 基石

当然这样的设想不是 `Hope` 首创， 已经有很多的先例在尝试这样的方式，定义和分享 Api, 比如; [gRPC-Gateway](https://github.com/grpc-ecosystem/grpc-gateway) 用以打通grpc 和 RESTful JSON API 之间的桥梁：

![](../public/image/protobuf/architecture_introduction_diagram.svg)

以及整个 google 系 API的 protobuf 定义 [Goole Open API Protos](https://github.com/googleapis/googleapis/tree/master/google/api)

## 参考

1. [gRPC-Gateway](https://github.com/grpc-ecosystem/grpc-gateway) ，gRPC-Gateway is a plugin of protoc. It reads a gRPC service definition and generates a reverse-proxy server which translates a RESTful JSON API into gRPC. 
2. [Goole Open API Protos](https://github.com/googleapis/googleapis/tree/master/google/api) ； public Google APIs that support both REST and gRPC protocols. 
3. [The OpenAPI Specification](https://github.com/OAI/OpenAPI-Specification)
