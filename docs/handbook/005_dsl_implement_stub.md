---
title: DSL - Stub - 实现
description: DSL 实现OAS OpenAPI Specification
---

[Wire](./004_dsl_implement_wire.md) 模块基本将全部的准备工作都做好， **Stub** 将生成实际引用的基础模板代码。

依然采用 “干湿” 分离的方式，生成代码和手写代码分别位于不同的 `sourceSet`。

## Meta

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

## Gradle

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

插件配置 `hopeStub`:

| 名称                | 说明                | 类型        | 值(默认) | 备注                                               |
|-------------------|-------------------|-----------|-------|--------------------------------------------------|
| debug             | 插件调试              | `boolean` | false | 配置检测完推出，不完成任何代码生成                                |
| verbose           | 执行过程log打开         | `boolean` | false | 出问题打开，过程全log打开调试                                 |
| generatedVersion  | 是否携带生成插件版本        | `boolean` | false | 生成代码 `@Generated` 说明是否携带插件版本                     |
| generatedTime     | 是否携带生成时间戳         | `boolean` | false | 生成代码 `@Generated` 说明是否携带生成时间戳                    |
| pluginMainVersion | 插件辅助版本            | `String`  |       | wire 插件版本运行时依赖版本，非必要勿设置，Apihug整体包BOM发行，勿手动设置这个版本 |
| pluginMainClass   | 插件辅助入口Main        | `String`  |       | 如非扩展了插件，勿设置                                      |
| enableFrontVue    | 是否生成Front vue 脚手架 | `boolean` | false | 实验阶段                                             |
| adminStub         | 是否生成Admin管理代码     | `boolean` | false | 未启用                                              |

刷新本模块的 gradle 后， 在task 列表可以看到 `stub` 命令被添加进来， 运行此模块下的 stub 命令 `gradlew stub`

1. `main\stub` 生成运行时代码
2. `main\resources\liquibase` 生成数据库迁移代码

## Intellj

## Refer

1. [The OpenAPI Specification](https://github.com/OAI/OpenAPI-Specification)