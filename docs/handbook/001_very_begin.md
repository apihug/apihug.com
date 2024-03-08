---
title: 快速开始
description: 开箱即用apihug, apihug 依赖包， apihug 命令，5分钟开始apihug。
---

😆 视频教程：

1. [ApiHug101-001开启篇-bilibili](https://www.bilibili.com/video/BV1KK421k7J8/)
2. [ApiHug101 Youtube](https://youtube.com/@ApiHug?si=C1yw0poHA01zbmyj)

## 版本信息

Apihug 包含组件

| 类别         | 说明            | 备注                                 |
|------------|---------------|------------------------------------|
| 核心包依赖      | 包含辅助类等非常薄一层扩展 | 核心包含运行时配置，可选应用层粘合剂                 |
| Gradle 插件  | 辅助DSL,代码生成    | 包含wire, proto编译时扩展， stub，应用层模板代码生成 |
| Intellj 插件 | 辅助设计,测试       | 可视化配置和设计(TBD)                      |

## 快速跑个例子

版本 [发行信息](../release/README.md)， 当前最新 <a target="_blank" href="https://search.maven.org/artifact/com.apihug/it-bom"><img src="https://img.shields.io/maven-central/v/com.apihug/it-bom.svg?label=Maven%20Central" /></a> (未到1.0.0前，版本升级比较快，注意通知)

💝 [最新版本查询](https://central.sonatype.com/search?q=com.apihug)， 注意， apihug 采用整包发行， plugin + 基础包公用一个版本。

建议采用 [gradle catalog](https://docs.gradle.org/current/userguide/platforms.html) + BOM 方式统一控制版本，以防版本冲突！

例子 [Hope Full Example Git](https://github.com/apihug/apihug-full-demo)

项目模板 + Intellj 项目入口， 设计进行中 🚧

### gradle 配置

项目 **settings.gradle**

```groovy

pluginManagement {
    repositories {
        mavenLocal()
        maven { url "https://repo.spring.io/libs-release" }
        maven { url "https://repo.spring.io/milestone" }
        mavenCentral()
        gradlePluginPortal()
    }
    plugins {

        //for proto job
        id 'com.google.protobuf' version "0.8.19"

        //for the local testing
        id 'com.apihug.wire' version '0.2.6-RELEASE'
        id 'com.apihug.stub' version '0.2.6-RELEASE'

    }
}
```

### Wire 配置

配置 proto 项目模块 `build.gradle`：

```goovy

plugins {
    id "java-library"
    id "com.apihug.wire"
}


hopeWire {
    verbose = true
}
```

设置项目元信息 `resources\hope-wire.json`:

```json
{
  "packageName": "hope.we",
  "name": "we-hope-server-next-proto",
  "application": "we-hope-server-next",
  "domain": "hope",
  "persistence": {
    "identifyType": "LONG",
    "tenantType": "LONG"
  },
  "createdAt": "2023-01-01",
  "createdBy": "Aaron"
}

```

在模块 `src/main/proto`下定义你的对象和RPC 接口后，刷新 gradle， 模块的 `task` 里多了  `wire` 等命令，
执行 `gradlew clean build` 项目生成：

1. `resources`， 生成配置信息
2. `main/wire`， 生成辅助代码

### Stub 配置

```groovy
plugins {
    id "java"

    id "org.springframework.boot"
    id "io.spring.dependency-management"

    id "com.apihug.stub"

    id "org.liquibase.gradle"
}


hopeStub {
    debug = false
}
```

设置项目元信息 `resources\hope-stub.json`:

```json
{
  "packageName": "hope.we",
  "name": "we-hope-server-next",
  "domain": "hope",
  "proto": {
    "name": "we-hope-server-next-proto",
    "module": "we-hope-server-next-proto",
    "applied": true,
    "version": "0.0.1"
  },
  "dependencies": [
  ]
}
```

刷新本模块的 gradle 后， 在task 列表可以看到 `stub` 命令被添加进来， 运行此模块下的 stub 命令 `gradlew stub`

1. `main\stub` 生成运行时代码
2. `main\resources\liquibase` 生成数据库迁移代码

## 核心

### Wire

Proto OAS 协议扩展解析编译插件 + 模板代码生成。

插件配置 `hopeWire`:

| 名称                              | 说明             | 类型        | (默认)  | 备注                                                   |
|---------------------------------|----------------|-----------|-------|------------------------------------------------------|
| disable                         | 插件调试           | `boolean` | false | 依赖注入完成，不添加额外 Task，调试测试用                              |
| smock                           | 插件调试           | `boolean` | false | 上面步骤完成 + Task 注入完毕， 不执行最后的生成动作，调试测试用                 |
| debug                           | 插件调试           | `boolean` | false | 上面步骤完成 + 执行生成动作第一阶段准备， 不最终触发动作，调试测试用                 |
| restrict                        | 非扩展类型是否隐式转换    | `boolean` | false | `google.protobuf.Timestamp` 定义是否支持默认转换到 local time   |
| verbose                         | 执行过程log打开      | `boolean` | false | 出问题打开，过程全log打开调试                                     |
| generatedVersion                | 是否携带生成插件版本     | `boolean` | false | 生成代码 `@Generated` 说明是否携带插件版本                         |
| generatedTime                   | 是否携带生成时间戳      | `boolean` | false | 生成代码 `@Generated` 说明是否携带生成时间戳                        |
| pluginMainVersion               | 插件辅助版本         | `String`  |       | wire 插件版本运行时依赖版本，非必要勿设置，Apihug整体包BOM发行，勿手动设置这个版本     |
| pluginMainClass                 | 插件辅助入口Main     | `String`  |       | 如非扩展了插件，勿设置                                          |
| local                           | 本地插件           | `boolean` | false | 依赖自己扩展插件，如不是，勿设置                                     |
| protocVersion                   | protoc版本       | `String`  |       | Wire发行时候自带，可以自行定义(可能导致不兼容风险)                         |
| grpcVersion                     | grpc版本         | `String`  |       | Wire发行时候自带，可以自行定义(可能导致不兼容风险)                         |
| keepProto                       | 是否包含proto原编译文件 | `boolean` | false | wire项目发行时候是否携带protoc编译结果，除项目间有深度逻辑依赖，共享的应局限在POJO对象这层 |
| wireProtoBufGradlePluginVersion | protobuf插件版本   | `String`  |       | 未启用                                                  |
| validationVersion               | Validation版本   | `String`  |       | 未启用                                                  |
| swaggerVersion                  | Swagger版本      | `String`  |       | 未启用                                                  |

项目配置元信息 `resources\hope-wire.json`：

| 名称                       | 说明        | 类型      | (默认) | 备注                                                             |
|--------------------------|-----------|---------|------|----------------------------------------------------------------|
| packageName              | 包名        | `Sting` | 必须   | 项目包名，符合java包命名规范，不可包含预留： `wire`, `stub` 关键字                    |
| name                     | 项目名称      | `Sting` | 必须   | 项目标识，符合 artifact ID, 小写，中文标识，proto后缀比如: user-info-proto        |
| application              | 应用项目      | `Sting` | 必须   | 和proto配套项目名称， 一般是name 去掉 proto 后缀比如： user-info                 |
| module                   | 模块名称      | `Sting` | 必须   | 用在运行时 service locator定位， 如无设置 `name`替代， domain+module需要保证运行时唯一 |
| domain                   | 领域名       | `Sting` | 必须   | 属于领域                                                           |
| description              | 描述        | `Sting` | 可选   | 描述                                                             |
| persistence.identifyType | 数据库，账号类型  | `Sting` | 可选   | 数据设计，有账号ID标识时启用, STRING\|INTEGER\|LONG                         |
| persistence.tenantType   | 数据库，租户类型  | `Sting` | 可选   | 数据设计，有多租户标识时启用, STRING\|INTEGER\|LONG                          |
| authority.enumClass      | 验权，权限类型枚举 | `Sting` | 可选   | 验权设计，权限枚举类，必须在 proto 有定义，编译时校验                                 |
| authority.codePrefix     | 验权，权限标识段  | `Long`  | 可选   | 验权设计，多模块下，权限标识代码分段，避免重复，比如100000 为`user`权限区间段,100001为第一个       |

### Stub

根据 Wire 结果生成，应用层代码脚手架。

插件配置 `hopeStub`:

| 名称                | 说明                | 类型        | (默认)  | 备注                                               |
|-------------------|-------------------|-----------|-------|--------------------------------------------------|
| debug             | 插件调试              | `boolean` | false | 配置检测完推出，不完成任何代码生成                                |
| verbose           | 执行过程log打开         | `boolean` | false | 出问题打开，过程全log打开调试                                 |
| generatedVersion  | 是否携带生成插件版本        | `boolean` | false | 生成代码 `@Generated` 说明是否携带插件版本                     |
| generatedTime     | 是否携带生成时间戳         | `boolean` | false | 生成代码 `@Generated` 说明是否携带生成时间戳                    |
| pluginMainVersion | 插件辅助版本            | `String`  |       | wire 插件版本运行时依赖版本，非必要勿设置，Apihug整体包BOM发行，勿手动设置这个版本 |
| pluginMainClass   | 插件辅助入口Main        | `String`  |       | 如非扩展了插件，勿设置                                      |
| enableFrontVue    | 是否生成Front vue 脚手架 | `boolean` | false | 实验阶段                                             |
| adminStub         | 是否生成Admin管理代码     | `boolean` | false | 未启用                                              |

项目配置元信息依赖 `resources\hope-stub.json`：

1. proto 配置自服务协议定义依赖， **单个**
2. dependencies三方服务协议定义依赖，理解为其他微服务依赖， **多个**
3. artifact部分如果在自 `build.gradle` 通过 `dependencies` 以各种方式已经引入，为非必要配置项，无法编译时候校验检测，如果配置错误，可能导致运行时出错！⚠️⚠️⚠️
    1. 服务协议 proto 包的依赖，可通过标准 gradle 依赖控制；
    2. 也可以通过在 `resources\hope-stub.json` 配置，插件帮你处理；
    3. 保持统一方式，勿混合使用，否则定位问题麻烦；

| 名称                               | 说明                   | 类型        | (默认)  | 备注                                                              |
|----------------------------------|----------------------|-----------|-------|-----------------------------------------------------------------|
| packageName                      | 包名                   | `Sting`   | 必须    | 项目包名，符合java包命名规范，不可包含预留： `wire`, `stub` 关键字                     |
| name                             | 项目名称                 | `Sting`   | 必须    | 项目标识，符合 artifact ID, 小写，中文标识，user-info, 为proto项目对应的application  |
| domain                           | 包名                   | `Sting`   | 必须    | 属于领域                                                            |
| proto.artifact.groupId           | 自服务proto group       | `Sting`   | 可选    | 自服务， 依赖包 group Id                                               |
| proto.artifact.artifactId        | 自服务proto artifactId  | `Sting`   | 可选    | 自服务， 依赖包 artifact Id                                            |
| proto.artifact.version           | 自服务proto 版本          | `Sting`   | 可选    | 自服务， 依赖包 版本                                                     |
| proto.module                     | 自服务proto 模块名称        | `Sting`   | 必须    | 用在运行时 service locator定位，**如未设置** `proto.artifact.artifactId` 替代 |
| proto.domain                     | 自服务proto 领域          | `Sting`   | 可选    | **如未设置**，和主项目一致                                                 |
| proto.applied                    | 自服务proto 是否已引入       | `boolean` | 必须    | 是否已经引入,如果已在 `gradle>dependencies`引入，不再插件引入lib                   |
| dependencies.artifact.groupId    | 三方服务proto group      | `Sting`   | 可选    | 三方服务， 依赖包 group Id                                              |
| dependencies.artifact.artifactId | 三方服务proto artifactId | `Sting`   | 可选    | 三方服务， 依赖包 artifact Id                                           |
| dependencies.artifact.version    | 三方服务proto 版本         | `Sting`   | 可选    | 三方服务， 依赖包 版本                                                    |
| dependencies.module              | 三方服务proto 模块         | `Sting`   | 必须    | 三方服务，模块名称，**如未设置**，和 `artifact.artifactId`保持一致                  |
| dependencies.domain              | 三方服务proto 领域         | `Sting`   | 必须    | 三方服务， 领域                                                        |
| dependencies.applied             | 三方服务proto 已引入否       | `boolean` | false | 是否已经引入， 如果已在 `gradle>dependencies`引入，不再插件引入lib                  |

### Core

| 类别                | 名称                 | 说明                                              |
|-------------------|--------------------|-------------------------------------------------|
| it-common         | 共享                 | 协议这部分的共享，无任何第三方项目和框架依赖                          |
| it-common-api     | 共享                 | 协议这部分的共享，和API定义描述相关，无三方依赖                       |
| it-common-spring* | 共享                 | 协议这部分的共享，和spring 相关部分，spring 核心依赖               |
| it-proto-extend   | proto buffer 扩展    | 纯扩展，包括在: 文件/message/enum/Service/method等上面元信息扩展 |
| it-plugin-wire    | wire gradle plugin | wire 这里gradle扩展， 辅助包，插件引入，最低化学习成本               |
| it-plugin-stub    | stub gradle plugin | stub 应用端的代码生成， spring 框架融入                      |

## Intellj

[IDEA Plugin 手册](../IDE/README.md)
