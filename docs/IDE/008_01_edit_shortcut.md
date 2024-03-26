---
title: IDE Plugin Proto 编辑快捷键
description: 编辑快捷键 shortcut
--- 

since `0.2.6`+

⚠️及时更新升级ApiHug 插件⚠️

💝 IDEA 插件： [ApiHug - API design Copilot](https://plugins.jetbrains.com/plugin/23534-apihug--api-design-copilot)

## 快捷键 shortcut

Preconditions:

1. 必须是 `proto` 类型文件;  must be `proto` kind file;
2. 文件 option 需要满足特定标识： 比如 `option (hope.domain.kind) = IDL` 为`IDL`接口类型文件; must mark with ApiHug extend optional， for example: `option (hope.domain.kind) = IDL` for `IDL` file;

| 快捷键 shortcut          |说明 Comment       | Tip        | 条件 Preconditions|
|-------------|------------|-----------|-----------|
| `ctl + alt +shift + M`     | 添加 对象 Add Message       | `M` for Message |`BEAN`|
| `ctl + alt +shift + F`     | 添加 字段 Add Field       | `F` for Field  |`BEAN`|
| `ctl + alt +shift + K`     | 添加 字段(引用) Add Field(Reference)       | `K` for ?  |`BEAN`|
| `ctl + alt +shift + U`     | 添加 枚举 Add Enum       | `U` for Unique constant  |`ENUM`|
| `ctl + alt +shift + X`     | 添加 枚举字段 Add Enum Field       | `X` for ?  |`ENUM`|
| `ctl + alt +shift + A`     | 添加 错误 Add Error       | `A` for ?  |`ERROR`|
| `ctl + alt +shift + Z`     | 添加 错误字段 Add Error Field      | `Z` for ?  |`ERROR`|
| `ctl + alt +shift + R`     | 添加 服务资源 Add Service      | `R` for resource  |`IDL`|
| `ctl + alt +shift + P`     | 添加 服务路径 Add Path      | `P` for path |`IDL`|
| `ctl + alt +shift + Y`     | 添加 实体 Add Entity      | `Y` for ?  |`ENTITY`|
| `ctl + alt +shift + O`     | 添加 实体字段 Add Entity Field      | `O` for ?  |`ENTITY`|

## Reference

1. [ApiHug101-Bilibili](https://space.bilibili.com/666522636)
2. [ApiHug101-Youtube](https://youtube.com/@ApiHug?si=C1yw0poHA01zbmyj)