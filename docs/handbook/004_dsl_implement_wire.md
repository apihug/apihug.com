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

| 名词          | 用途           | 说明     |
|-------------|--------------|--------|
| tag         | 服务标签         | 对象Tag  |
| path        | 服务基础前缀，默认`/` | string |
| description | 服务描述         | string |


### Method

`hope.swagger.operation` 定义方法元信息扩展：

```proto
extend google.protobuf.MethodOptions {
    Operation operation = 1042;
}
```

`MethodOptions`  operation 描述，  [Operation 对象](https://swagger.io/specification/#operation-object)

| 字段           | 类型                        | 说明                                                                        |
|--------------|---------------------------|---------------------------------------------------------------------------|
| request_name | 输入参数命名                    | 单 input 对象在函数中的命名，如果有parameter说明这个不需要，否则会自动推算 `_0(1,2,3)` 这样子比较丑，帮助代码生成器用 |
| priority     | Priority 枚举               | devops 流程控制，[参考](#priority-枚举类型)                                          |
| pageable     | 是否支持pageable              | 入参有PageRequest, 结果自动分页包装 [参考](#oas-hope-page)                             |
| raw          | 是否原始值                     | 如非原始包装，会统一Result 风格封装                                                     |
| request      | 是否带原始的request             | 比如servlet 协议手动处理                                                          |
| response     | 是否带原始的response            | 比如servlet协议手动处理                                                           |
| session      | 是否带session                | 视具体框架实现                                                                   |
| input_plural | 输入是数组                     | 输入对象数组包装 List, 避免proto 对象定义爆炸                                             |
| out_plural   | 输出是数组                     | 输出对象数组包装 List， 避免proto 对象定义爆炸                                             |
| pattern      | get/put/post/delete/patch | 目前只支持者几个对象 oneof， http action 语义                                          |
| parameters   | 参数对象                      | 是一个 [Parameter](#parameter-参数对象) 数组对象                                     |

#### Priority 枚举类型

开发流程审批规范：

| 字段       | 类型    | 说明      |
|----------|-------|---------|
| LOW      | 重要度低  | 开发者交叉审批 |
| MIDDLE   | 重要度中等 | 开发组长审批  |
| HIGH     | 重要度高  | 项目管理者审批 |
| CRITICAL | 重要度很高 | 总监审批    |
| FATAL    | 重要度致命 | CTO审批   |

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

| 字段     | 类型         | 说明                                                    |
|--------|------------|-------------------------------------------------------|
| name   | 参数名词       | 写标准点， 转换成方法字段会自动推理， 比如 `hello-world` --> `helloWorld` |
| in     | 参数类型       | 参考下面枚举                                                |
| scheme | JSONSchema | [JSONSchema](#jsonschema)                             |
| plural | 是否列表       | boolean                                               |

#### `IN` 参数

类型  [OAS 参数对象](https://swagger.io/specification/#parameter-object)

| 字段      | 类型           | 说明                                              |
|---------|--------------|-------------------------------------------------|
| QUERY   | parameter    | `path/arg1=123`, arg1 参数                        |
| HEADER  | HTTP 协议头     | 比如JWT, 或者扩展字段                                   |
| PATH    | 路径           | `/path/{arg}/hello`, `arg` 记得必须是 `{}`, 目前没有强制校验 |
| COOKIE  | 来自cookie 参数  | 视底层框架                                           |
| SESSION | 来自session 参数 | 视底层框架                                           |

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

扩展字段：

| 字段                     | 类型                                   | 说明                                                                                        |
|------------------------|--------------------------------------|-------------------------------------------------------------------------------------------|
| description            | `string`                             | 描述                                                                                        |
| default                | `string`                             | 默认值                                                                                       |
| example                | `string`                             | 示例                                                                                        |
| multiple_of            | `google.protobuf.DoubleValue`        | 倍数                                                                                        |
| maximum                | `google.protobuf.DoubleValue`        | 最大值                                                                                       |
| exclusive_maximum      | `google.protobuf.BoolValue`          | 区间是否包含最大值                                                                                 |
| minimum                | `google.protobuf.DoubleValue`        | 最小值                                                                                       |
| exclusive_minimum      | `google.protobuf.BoolValue`          | 区间是否包含最小值                                                                                 |
| max_length             | `google.protobuf.UInt64Value`        | 最大长度，string类型                                                                             |
| min_length             | `google.protobuf.UInt64Value`        | 最小长度，string类型                                                                             |
| max_items              | `google.protobuf.UInt64Value`        | `repeated` 集合类型字段，最多元素数目                                                                  |
| min_items              | `google.protobuf.UInt64Value`        | `repeated` 集合类型字段，最多元素数目                                                                  |
| unique_items           | `google.protobuf.BoolValue`          | `repeated` 集合类型字段，元素是否唯一，`List vs Set`                                                    |
| ~type~                 | ~`JSONSchemaTypeHint`~               | 😢                                                                                        |
| format                 | `string`                             | [JSONSchemaFormat](#jsonschemaformat)                                                     |
| field_configuration    | `FieldConfiguration`                 | ✋ [参考](#fieldconfiguration)                                                               |
| empty                  | `google.protobuf.BoolValue`          | 是否可以为空-校验                                                                                 |
| pattern                | `string`                             | validation 扩展:正值表达式验证 `javax.validation.constraints.Pattern("^A-z$")`                     |
| assert                 | `google.protobuf.BoolValue`          | validation 扩展: `javax.validation.constraints.AssertTrue\AssertFalse`                      |
| decimal_max            | `string`                             | validation 扩展:`javax.validation.constraints.DecimalMax(value = "0.0", inclusive = false)` |
| decimal_min            | `string`                             | validation 扩展:`javax.validation.constraints.DecimalMin(value = "0.0", inclusive = false)` |
| digits_integer         | `google.protobuf.Int32Value`         | validation 扩展:`javax.validation.constraints.Digits(integer=3, fraction=2)`                |
| digits_fraction        | `google.protobuf.Int32Value`         | validation 扩展:`javax.validation.constraints.Digits(integer=3, fraction=2)`                |
| email                  | `google.protobuf.BoolValue`          | validation 扩展:`javax.validation.constraints.Email`                                        |
| time_constraint_type   | `TimeConstraintType`                 | validation 扩展:枚举参考下面 [TimeConstraintType](#timeconstrainttype)                            |
| date_format            | `DateFormat`                         | 日期format: 日期枚举类型参考下面 [DateFormat](#dateformat-枚举类型)                                       |
| customized_date_format | `string`                             | 定制日期类型:符合标准日期定义规范(未强校验)                                                                   |
| mock                   | `Mock`                               | Mock规则 🏗️                                                                                |
| read_only              | `bool`                               | 未用 🚧                                                                                     |
| extensions             | `map<string, google.protobuf.Value>` | 未用 🚧，扩展说明                                                                                |
| enum                   | `repeated string`                    | 未用 🚧                                                                                     |
| required               | `repeated string`                    | 未用 🚧， 范围选择，通过枚举对象实现                                                                      |
| array                  | `repeated string`                    | 未用 🚧， 列表元素可选范围，通过枚举对象实现                                                                  |
| ref                    | `string`                             | 未用 🚧, 外部对象引用，需全路径指定 **parameter配置时，如引用 Enum 对象**                                         |
| title                  | `string`                             | 未用 🚧, 标题, 字段名称替代                                                                         |
| ~max_properties~       | `google.protobuf.UInt64Value`        | ~未用 🚧，Map元素最多key?~                                                                       |
| ~min_properties~       | `google.protobuf.UInt64Value`        | ~未用 🚧，Map元素最多key?~                                                                       |

⚠️ 由于框架层引入常量设计机制， 所以很多需要通过 `enum`, `required`, `array` 控制的，通过枚举控制均都可弱化和替代掉。

#### FieldConfiguration

```proto
message FieldConfiguration {
        string path_param_name = 47;
}
```

在[Parameter 参数对象](#parameter-参数对象) 中定义灵活对象时， 在路径参数上对象和宿主语言命名有时候有冲突；
比如 参数 `{user-id}` 在java 中无法使用 `user-id` 命名参数， 中划线 `-` 为字段命名非法字符，如果不设定，会`隐式`的转换成 `userId`, 但是同时也可以自己定义：

```proto
field_configuration: {
    path_param_name: "orderIdAnother"
}
```

#### JSONSchemaFormat

为避免复杂度， ~JSONSchemaTypeHint~ 自 **0.3.3.RELEASE** 被合并到 `format`

和 `format` 合并了[OpenApi format](https://spec.openapis.org/oas/latest.html#format)  (OpenAPI Specification v3.1.0,2023-12) + 部分自有扩展。

一般 Message 字段定义已经包含 `显式` 类型定义： 内置类型或者引用类型， 但是在 `option` 里的定义无法设定，比如在[Parameter 参数对象](#parameter-参数对象)。

所以需要这里 `隐式` 的制定类型， 或者需要强制将内置类型对象转换成语言特定对象， 比如 `string` `隐式` 成一个 `DateTime`， 方便代码生成器推导宿主语言对象类型。

| 字段          | 类型         | 说明        | OAS format  |
|-------------|------------|-----------|-------------|
| BOOLEAN     | bool       | 类型boolean | bool        |
| INTEGER     | integer    | 整型        | int32       |
| DOUBLE      | 数字         | double    | double      |
| STRING      | string     | 字符串       | -           |
| FLOAT       | float      | 浮点类型      | float       |
| BIG_DECIMAL | bigDecimal | 精度数字      | big-decimal |
| LONG        | long       | 长整型       | int64       |
| DATE        | date       | 日期        | date        |
| DATE_TIME   | dateTime   | 日期时间      | date-time   |
| TIME        | time       | 时间        | time        |
| UUID        | uuid       | UUID 对象   | uuid        |
| PASSWORD    | password   | 密码对象      | password    |
| EMAIL       | email      | 邮箱对象      | email       |
| BINARY      | binary     | 文件对象      | binary      |

⚠️ 谨慎使用，这也是为什么我们推崇，复杂对象通过 `Message` 定义， 避免这种 `Ad-hoc` Path/Query/Header `隐式`推断；
固然通过强制 仅支持 `post`可以尽量避免， 但是 `ApiHug` 还是兼容了这些古老的做法。

#### TimeConstraintType

| 字段                | 类型   | 说明                                              |
|-------------------|------|-------------------------------------------------|
| FUTURE            | 时间校验 | `@javax.validation.constraints.Future`          |
| FUTURE_OR_PRESENT | 时间校验 | `@javax.validation.constraints.FutureOrPresent` |
| PAST              | 时间校验 | `@javax.validation.constraints.Past`            |
| PAST_OR_PRESENT   | 时间校验 | `@javax.validation.constraints.PastOrPresent`   |

#### DateFormat 枚举类型

预定义formatter [java.time.format.DateTimeFormatter](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html)

或自定义  `customized_date_format`, 最好用预定义的， 通过了校验测试。

如果使用 `customized_date_format` 推导宿主语言类型有不确定性⚠️， 而预定义可以分化： `DateTime`, `Time`, `Date` 更细的分类 ⭐⭐⭐。

| 字段                            | 类型                      | 说明                                          |
|-------------------------------|-------------------------|---------------------------------------------|
| BASIC_ISO_DATE                | yyyyMMdd                | 比如 `20111203` 类型 `LocalDate`                |
| ISO_LOCAL_DATE                | yyyy-MM-dd              | 比如 `2011-12-03` 类型 `LocalDate`              |
| ISO_LOCAL_TIME                | HH:mm:ss                | 比如 `10:15:30` 类型 `LocalTime`                |
| ISO_LOCAL_DATE_TIME           | yyyy-MM-dd T HH:mm:ss   | 比如 `2011-12-03T10:15:30` 类型 `LocalDateTime` |
| YYYY_MM_DD_HH_MM_SS           | yyyy-MM-dd HH:mm:ss     | 类型 `LocalDateTime`                          |
| YYYY_MM_DD_HH_MM_SS_SSS       | yyyy-MM-dd HH:mm:ss:SSS | 类型 `LocalDateTime`                          |
| SLASH_YYYY_MM_DD              | yyyy/MM/dd              | 类型 `LocalDate`                              |
| SLASH_YYYY_MM_DD_HH_MM_SS     | yyyy/MM/dd HH:mm:ss     | 类型 `LocalDateTime`                          |
| SLASH_YYYY_MM_DD_HH_MM_SS_SSS | yyyy/MM/dd HH:mm:ss:SSS | 类型 `LocalDateTime`                          |
| HH_MM                         | HH:mm                   | 类型 `LocalTime`                              |

自定义格式日期时间判断方式：

1. Date 判断包含任何一个：
   1. yyyy
   2. MM
   3. dd
2. Time 判断包含任何一个：
   1. HH
   2. mm
   3. ss
   4. SSS

### Enum

`Hope` 框架里，引入了 枚举常量的设计， 大大减少了常量的硬编码， `Enum` 在宿主语言 `Java`, `c`, `Go` 等都有完整的支持。

同时在 `Enum` 上又扩展了：错误码 `Error` + `Authority`类型。

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
message Meta {

  int32 code = 1;
  string message = 2;
  string cn_message = 3;

  Error error = 4;
}
```

| 名词 | 用途 | 说明 |
| --- | --- | --- |
|code| 错误代码 | int |
|message|错误信息| string|
|cn_message|错误信息(中)| string|

#### Error

错误枚举，在枚举上扩展 `Error error = 4;` 字段包含;

| 字段 | 类型 | 说明 |
| --- | --- | --- |
|title|`string`|简单标题|
|tips|`string`|提示|
|http_status|`HttpStatus`|Http 错误码|
|phase|`Phase`|阶段,[参考下面文档](#错误阶段)|
|severity|`Severity`|严重程度,[参考下面文档](#错误严重性)|

##### 错误阶段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
|CONTROLLER|Controller|表单层|
|SERVICE|Service|服务层|
|DOMAIN|Domain|领域层|

#### 错误严重性

| 字段 | 类型 | 说明 |
| --- | --- | --- |
|LOW|低|低,无影响|
|WARN|警告|警告,业务错误可重试|
|ERROR|错误|错误,业务无法进行|
|FATAL|灾难|灾难,数据破坏|

#### Authority

Authority 为标准 `Enum` 类型， 在 `Wire`项目的元信息中指定 [authority meta](./001_very_begin.md#wire-配置)：

```json
 "authority" : {
    "enumClass" : "hope.we.proto.infra.enumeration.authority.SystemAuthorityEnum",
    "codePrefix" : 11000000
  }
```

#### Map

对于Map类型字段，处理比较头疼， 如果混合宿主语言特征，会非常棘手， 比如 key, value 都有 `Generic` 特性。
综合：**OAS** [Dictionaries, HashMaps and Associative Arrays](https://swagger.io/docs/specification/data-models/dictionaries/), Protobuf: [Maps Features](https://protobuf.dev/programming-guides/proto3/#maps-features) 如下约束：

1. Key, Value 不能 `Generic` 也就是只能固定类型， 宿主语言内的变幻， 无法控制，比如子类判断。
2. 参考 [Protobuf Map](./002_protobuf_basic.md#map) 将 key, value 转换成独立 `Message` 对象。

## Meta

项目基本元信息 `resources\hope-wire.json`：

```json
{
  "packageName": "hope.we",
  "name": "bigger-project-proto",
  "application": "bigger-project",
  "domain": "hope",
  "persistence" : {
    "identifyType" : "LONG",
    "tenantType" : "LONG"
  },
  "authority" : {
    "enumClass" : "hope.we.proto.infra.enumeration.authority.SystemAuthorityEnum",
    "codePrefix" : 11000000
  },
  "createdAt": "2023-01-01",
  "createdBy" : "Aaron",
  "api": {
    "openapi" : "3.0.1",
    "info" : {
      "contact" : {
        "name" : "developer@apihug.com",
        "url" : "https://github.com/hope/apihug",
        "email" : "developer@apihug.com"
      }
    },
    "externalDocs" : {
      "description" : "Hope is the best thing",
      "url" : "https://github.com/hope/apihug/"
    },
    "tags" : [ {
      "name" : "tenant",
      "description" : "租户操作"
    }, {
      "name" : "platform",
      "description" : "平台操作"
    }]
  }
}
```

| 名称                       | 说明        | 类型 | 值(默认)     | 备注                                                             |
|--------------------------|-----------|------------|------------|----------------------------------------------------------------|
| packageName              | 包名        | `Sting`|必须 | 项目包名，符合java包命名规范，不可包含预留： `wire`, `stub` 关键字                    |
| name                     | 项目名称      | `Sting`|必须 | 项目标识，符合 artifact ID, 小写，中文标识，proto后缀比如: user-info-proto        |
| application              | 应用项目      | `Sting`|必须 | 和proto配套项目名称， 一般是name 去掉 proto 后缀比如： user-info                 |
| module                   | 模块名称      | `Sting`|必须 | 用在运行时 service locator定位， 如无设置 `name`替代， domain+module需要保证运行时唯一 |
| domain                   | 领域名        | `Sting`|必须 | 属于领域                                                           |
| description              | 描述        | `Sting`|可选 | 描述                                                             |
| persistence.identifyType | 数据库，账号类型  | `Sting`|可选 | 数据设计，有账号ID标识时启用, STRING\|INTEGER\|LONG                         |
| persistence.tenantType   | 数据库，租户类型  | `Sting`|可选 | 数据设计，有多租户标识时启用, STRING\|INTEGER\|LONG                          |
| authority.enumClass      | 验权，权限类型枚举 | `Sting`|可选 | 验权设计，权限枚举类，必须在 proto 有定义，编译时校验                                 |
| authority.codePrefix     | 验权，权限标识段  | `Long`|可选  | 验权设计，多模块下，权限标识代码分段，避免重复，比如100000 为`user`权限区间段,100001为第一个       |

### OAS

只包含基本的 OAS 说明：

1. `openapi` OAS 兼容 [版本](https://swagger.io/specification/#fixed-fields)
2. `info` OAS 基本 [Info](https://swagger.io/specification/#info-object)
3. `tag` OAS 标签 [Tag](https://swagger.io/specification/#tag-object)

⚠️其他信息，请勿这里定义，定义也会被忽略！

## 文件上传

协议定义✏️：

1. 方法：POST (必须)
2. consumes 类型： `multipart/form-data` （必须）

```proto
  option (hope.swagger.operation) = {
    post: "/upload-meta";
    consumes: "multipart/form-data";
  };
```

字段定义 `format: "binary"`:

```proto
  // Binary format meaning a file.
  string file = 5 [(hope.swagger.field) = {
    format: "binary"
  }];

  repeat bytes files = 6;
```

支持单文件， 多文件批量上传；

1. `format: "binary"` 自身类型 `string` 将被忽略；
2. `bytes` 类型，自动被识别为  `binary` 文件类型；
3. 支持 `repeat` 多文件批量上传；
4. 支持附带属性。

```proto
message SampleUploadRequest {
  option (hope.swagger.schema) = {
    json_schema: {
      description: "An example to upload a single file";
    };
  };

  string name = 4 [(hope.swagger.field) = {
    description: "new name of this upload file";
    example: "just_another_file"
    max_length: {
      value: 64
    }
  }];


  // Binary format meaning a file.
  string an_file = 5 [(hope.swagger.field) = {
    format: "binary"
  }];
}
```

## Command

配置 wire 项目模块 `build.gradle`：

```goovy

plugins {
    id "java-library"
    id "com.apihug.wire"
}


hopeWire {
    verbose = true
}
```

刷新本模块的 gradle 后， 在task 列表可以看到 `wire` 命令被添加进来， 运行此模块下的 wire 命令 `gradlew wire`

1. `main\wire` 生成运行时代码
2. `main\resources` 生成编译元信息

| 名称                              | 说明             | 类型|(默认)           | 备注                                                   |
|---------------------------------|----------------|------------------|------------|------------------------------------------------------|
| disable                         | 插件调试           | `boolean`|false  | 依赖注入完成，不添加额外 Task，调试测试用                              |
| smock                           | 插件调试           | `boolean`| false | 上面步骤完成 + Task 注入完毕， 不执行最后的生成动作，调试测试用                 |
| debug                           | 插件调试           | `boolean`| false | 上面步骤完成 + 执行生成动作第一阶段准备， 不最终触发动作，调试测试用                 |
| restrict                        | 非扩展类型是否隐式转换    | `boolean`| false | `google.protobuf.Timestamp` 定义是否支持默认转换到 local time   |
| verbose                         | 执行过程log打开      | `boolean`| false | 出问题打开，过程全log打开调试                                     |
| generatedVersion                | 是否携带生成插件版本     | `boolean`| false | 生成代码 `@Generated` 说明是否携带插件版本                         |
| generatedTime                   | 是否携带生成时间戳      | `boolean`| false | 生成代码 `@Generated` 说明是否携带生成时间戳                        |
| pluginMainVersion               | 插件辅助版本         | `String` |        | wire 插件版本运行时依赖版本，非必要勿设置，Apihug整体包BOM发行，勿手动设置这个版本     |
| pluginMainClass                 | 插件辅助入口Main     | `String` |        | 如非扩展了插件，勿设置                                          |
| local                           | 本地插件           | `boolean`|false  | 依赖自己扩展插件，如不是，勿设置                                     |
| protocVersion                   | protoc版本       | `String`|         | Wire发行时候自带，可以自行定义(可能导致不兼容风险)                         |
| grpcVersion                     | grpc版本         | `String`|         | Wire发行时候自带，可以自行定义(可能导致不兼容风险)                         |
| keepProto                       | 是否包含proto原编译文件 | `boolean`|false  | wire项目发行时候是否携带protoc编译结果，除项目间有深度逻辑依赖，共享的应局限在POJO对象这层 |
| wireProtoBufGradlePluginVersion | protobuf插件版本   | `String`|         | 未启用                                                  |
| validationVersion               | Validation版本   | `String`|         | 未启用                                                  |
| swaggerVersion                  | Swagger版本      | `String`|         | 未启用                                                  |

## Refer

1. [The OpenAPI Specification](https://github.com/OAI/OpenAPI-Specification)
2. [OpenAPI Specification](https://swagger.io/specification/)  Version 3.1.0 since 2023.
3. [Spring file upload](https://www.baeldung.com/spring-file-upload)
4. [ApiHug101-Bilibili](https://www.bilibili.com/video/BV1KK421k7J8/)
5. [ApiHug101-Youtube](https://youtube.com/@ApiHug?si=C1yw0poHA01zbmyj)
