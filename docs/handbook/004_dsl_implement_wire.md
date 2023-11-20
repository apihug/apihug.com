---
title: DSL - Wire 实现
description: DSL 实现 OAS OpenAPI Specification
---

Protobuf 定义 OpenAPI。

## Api

Service 概念对应 GRPC 中 Service 供应方。 方法对应具体服务， 对应 OAS中的[Paths Object](https://swagger.io/specification/#paths-object)

### Service

`hope.swagger.svc` 定义服务元信息扩展：

```proto
extend google.protobuf.ServiceOptions {

    ServiceSchema svc = 1044;
}

message ServiceSchema {
    //The tag of this service
    Tag tag = 1;

    //The base path of this service
    string path = 2;

    // Description of this Service
    string description = 1044;
}
```

| 名词 | 用途 | 说明 |
| --- | --- | --- |
|tag| 服务标签 | 对象Tag |
|path| 服务基础前缀，默认`/` | string|
|description| 服务描述| string|


### Method

`hope.swagger.operation` 定义方法元信息扩展：

```proto
extend google.protobuf.MethodOptions {
    Operation operation = 1042;
}
```

`MethodOptions`  operation 描述，  [Operation 对象](https://swagger.io/specification/#operation-object)

| 字段 | 类型 | 说明 |
| --- | --- | --- |
|request_name|输入参数命名|单 input 对象在函数中的命名，如果有parameter说明这个不需要，否则会自动推算 `_0(1,2,3)` 这样子比较丑，帮助代码生成器用|
|priority|Priority 枚举| devops 流程控制，[参考](#priority-枚举类型)|
|pageable|是否支持pageable|入参有PageRequest, 结果自动分页包装 [参考](#oas-hope-page)|
|raw|是否原始值|如非原始包装，会统一Result 风格封装|
|request|是否带原始的request|比如servlet 协议手动处理|
|response|是否带原始的response|比如servlet协议手动处理|
|session|是否带session|视具体框架实现|
|input_plural  |输入是数组|输入对象数组包装 List, 避免proto 对象定义爆炸|
|out_plural|输出是数组|输出对象数组包装 List， 避免proto 对象定义爆炸|
|pattern|get/put/post/delete/patch| 目前只支持者几个对象 oneof， http action 语义|
|parameters|参数对象| 是一个 [Parameter](parameter-参数对象) 数组对象|

#### Priority 枚举类型

开发流程审批规范：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
|LOW|重要度低|开发者交叉审批|
|MIDDLE|重要度中等|开发组长审批|
|HIGH|重要度高|项目管理者审批|
|CRITICAL|重要度很高|总监审批|
|FATAL|重要度致命|CTO审批|

#### Parameter 参数对象

⚠️  `0.2.6-RELEASE` 以后版本： 如果是 `GET` 类型

1. 禁止 input type 定义非引用对象， 复杂对象一律通过引用对象定义
2. 预定义对象(number/string/date 等)可以通过 parameters 定义

否则在 `wire` 阶段会报错：

```shell

proto illegal define @service.proto  TestService#GetUserInfo

It is a GET action, only support [google.protobuf.Empty] input or an reference Object while we got: google.protobuf.IntValue

all primitive types(int,long,date etc) please define in the [parameters] part, otherwise may mess the parameter judge!

```

因为在 input 定义， 然后又在 parameters 定义(没有语法错误)， 导致难判断， 到底用那个?

在非 `post` 传递, 非 `body` 对象时候使用,

1. RPC 定义 input 最好是个 `Empty`, 所有参数都在 `hope.swagger.operation#parameters` 中定义（⚠️最标准做法⚠️）
2. 如果是 `Message` 这个 `Message` 也会被当做 `parameter` 而非  `RequestBody`!
3. 可以承接多个对象， 
   1. 可以是原始对象： int/string/date/uuid 等，
   2. 或者是封装对象  `Message`, `Enum`

```js
rpc DeleteOrder (google.protobuf.Empty) returns (google.protobuf.StringValue) {
        option (hope.swagger.operation) = {
            post: "/delete-order";
            description: "删除订单",
            tags: "order";
            priority: CRITICAL;
            parameters: {
                parameter: {
                    name: "order-id";
                    in: QUERY;
                    scheme: {
                        type: STRING;
                        empty: {
                            value: false;
                        }
                        field_configuration: {
                            path_param_name: "orderId"
                        }
                    }
                }
            }
        };

};
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
|name|参数名词|写标准点， 转换成方法字段会自动推理， 比如 `hello-world` --> `helloWorld`|
|in|参数类型| 参考下面枚举|
|scheme|JSONSchema|[JSONSchema](#jsonschema)|
|plural|是否列表|boolean|

#### `IN` 参数

类型  [OAS 参数对象](https://swagger.io/specification/#parameter-object)

| 字段 | 类型 | 说明 |
| --- | --- | --- |
|QUERY|parameter|`path/arg1=123`, arg1 参数|
|HEADER |HTTP 协议头|比如JWT, 或者扩展字段|
|PATH|路径|`/path/{arg}/hello`, `arg` 记得必须是 `{}`, 目前没有强制校验|
|COOKIE|来自cookie 参数|视底层框架|
|SESSION|来自session 参数|视底层框架|

## Resource

### Message

`hope.swagger.schema` 为 `MessageOptions` 用来描述类型对象： [OAS Schema对象](https://swagger.io/specification/#schema-object)

```proto
extend google.protobuf.MessageOptions {
    Schema schema = 1042;
}
message QueryTenantRoleRequest {
  option (hope.swagger.schema) = {
    json_schema: {
      description: "查询角色列表";
    };
  };
}
```

消息整体的描述， 侧重消息的基本描述 `description`, `example`, `discriminator`,`read_only`, `external_docs` 当前未做解析 ⚠️。

### JSONSchema

字段描述 `hope.swagger.field` 为 `FieldOptions`， 扩展 `JSONSchema` 上面：

```proto
extend google.protobuf.FieldOptions {
    JSONSchema field = 1042;
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
|assert| validation 扩展| `javax.validation.constraints.AssertTrue\AssertFalse`|
|decimal_max|validation 扩展|`javax.validation.constraints.DecimalMax(value = "0.0", inclusive = false)`|
|decimal_min|validation 扩展|`javax.validation.constraints.DecimalMin(value = "0.0", inclusive = false)`|
|digits_integer|validation 扩展|`javax.validation.constraints.Digits(integer=3, fraction=2)`|
|digits_fraction|validation 扩展|`javax.validation.constraints.Digits(integer=3, fraction=2)`|
|email|validation 扩展|`javax.validation.constraints.Email`|
|time_constraint_type|validation 扩展|枚举参考下面 [TimeConstraintType](#oas-schema-tct)|
|date_format|日期format| 日期枚举类型参考下面 [DateFormat](#oas-schema-dt)|
|customized_date_format|定制日期类型|符合标准日期定义规范(未强校验)|

### Enum

`hope.swagger.enm` 为 `EnumOptions` 用来描述`常量`对象：

```proto
extend google.protobuf.EnumOptions {
    JSONSchema enm = 1042;
}
```

`hope.constant.field` 为 `EnumValueOptions` 用来描述`常量值`对象:

```proto
extend google.protobuf.EnumValueOptions {
  Meta field = 37020;
}
```

#### Authority

#### Error

## Meta

## Refer

1. [The OpenAPI Specification](https://github.com/OAI/OpenAPI-Specification)
2. [OpenAPI Specification](https://swagger.io/specification/)  Version 3.1.0 since 2023.
