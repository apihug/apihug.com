---
title: IDE 知识库
description: 知识库管理
---

## 0.5.4 upgrade

[014 Knowledge Settings](https://apihug.com/docs/idea/014-settings-knowledge) new web site;

Since IDEA Plugin `0.5.4`+  we move maintain of the knowledge as Project framework meta data, so you:

1. need to keep it in project source code to leverage the version control;
2. each module(domain) will has its own domain knowledge base;

The knowledge base configuration will host `{WIRE_MODULE}/src/main/resources/hope-knowledge.json`, please refer to this [sample](https://github.com/apihug/apihug-demo/blob/main/apihug-demo-proto/src/main/resources/hope-knowledge.json).

```json
{
  "knowledge" : [ {
    "name" : "user_name",
    "description" : "name of user",
    "attributeType" : "STRING",
    "notNull" : false,
    "notEmpty" : false,
    "notBlank" : true,
    "pattern" : "",
    "minLength" : 1,
    "maxLength" : 64,
    "sample" : "jake",
    "trackId" : "04b01f17-dc5b-44ce-b7ad-da7744dae723",
    "usage" : "BOTH"
  } ],
  "name" : "apihug-demo-proto",
  "application" : "apihug-demo",
  "packageName" : "com.apihug.sample",
  "domain" : "sample",
  "identifier" : "apihug-demo-proto",
  "module" : "apihug-demo-proto"
}
```

All other behavior keep same!

---

⚠️⚠️ After IDEA Plugin `0.2.6`+

⚠️Decoded after 0.5.4⚠️

**SDK**: <a target="_blank" href="https://search.maven.org/artifact/com.apihug/it-bom"><img src="https://img.shields.io/maven-central/v/com.apihug/it-bom.svg?label=Maven%20Central" /></a>  

**Plugin**: [ApiHug - API design Copilot](https://plugins.jetbrains.com/plugin/23534-apihug--api-design-copilot)

if your plugin still 0.1* please follow this: [0.1->0.2* plugin migration](../versions/001-milestone.md) to upgrade firstly!

## 什么是知识库

🎁 **团队共享设计标准共识**, **Team-shared design standards consensus**

1. 设计标准
2. 领域知识
3. 团队共识

## 配置

![KB](../public/image/idea/050_kb_01.png)

1. `Topbar>ApiHug>Knowledge`
2. Actions
   1. Add:  Add new knowledge
   2. Delete: Remove a knowledge
   3. Download Template: download Excel template for edit by batch  [中文模板](../public/image/idea/template/cn.xlsx)， [English template](../public/image/idea/template/en.xlsx)
   4. Import from template: import a modified excel/json file
   5. Export for share: export current knowledge for share with other members
3. Double click table row for modify!

![KB2](../public/image/idea/050_kb_02.png)

1. type for auto-complete

### 模板

1. 快速构建
2. 有限支持
3. json vs excel

### 共享

1. 导入
2. 导出

**Merger** 规则。

## 使用

1. Add Field
2. Auto complete (TBD)

## Reference

1. [ApiHug101-Bilibili](https://space.bilibili.com/666522636)
2. [ApiHug101-Youtube](https://youtube.com/@ApiHug?si=C1yw0poHA01zbmyj)
