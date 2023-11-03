---
title: ApiHug 简介
description: ApiHug 是个一个集 API 设计, 开发, 管理, 协同,CI/CD, 适合所有人的Api平台。
---

ApiHug 是一个开源，集设计、开发、管理、协同、CI/CD一体，适合所有人的API平台。

ApiHug 提供云上工作台，随时随地高效团队协同； 一站式工具链，协助你的团队快速构建高质量，高一致API平台。

![ApiHug](/images/apihug-platform.svg)

> OAS + DSL + Gradle + Framework + Ops 为卓越API团队生而。

如Java为企业开发提供基础，ApiHug为卓越API团队提供方案底座， ApiHug 不仅仅提供API设计协同平台， 更提供API开发实现、工具链、CI/CD集成，APM运行时监控管理。 

ApiHug 设计之初，团队时刻保持 [同理心](/principles/why-empathy-is-important), 不仅限于设计，到实现、部署、运维，平衡内部成本等都进行深度考量。

实现从业务方、产研方、管理者、出资人、用户等多方平衡， 从而提供一个高效、高质量、高一致的API解决方案平台。

## 核心功能

### [OAS开放API协议 DSL](/docs/sql-review/overview/)

ApiHug analyzes SQL changes to enforce rules in compliance with your organization's policy. The enforcement includes naming conventions, anti-SQL pattern detection and etc. Prod and non-prod environments can also enforce different rules respectively.

### [API全周期管理](/docs/change-database/change-workflow/)

Like code review, Bytebase streamlines the database change process. Within a single workflow, a database change can be reviewed and deployed from the dev environment all the way to the production environment.

### [团队协同](/docs/sql-editor/overview/)

A web-based SQL Editor to query and export data. DBAs no longer need to give away sensitive database credentials when Developers need to access the data.

### [管理审计](/docs/security/data-query/)

ApiHug provides a suite of features to enable organizations to enforce data security policies, avoid data leaks and conform compliance.

### [开发实现](/docs/vcs-integration/overview/)

ApiHug keeps the complete schema change history. It also integrates with VCS systems. Teams can manage the SQL migration scripts in the VCS and trigger schema deployment on code commit.

### [一站工具链](/docs/change-database/rollback-data-changes/)

- Statement-level rollback

- Database-level manual and periodical backup and restore

- Point-in-time recovery (PITR)

### [开放集成](/docs/change-database/rollback-data-changes/)

## 对比

### 平台对比

If Liquibase, Flyway are Git, then Bytebase is GitLab/GitHub. And as an open source project. Bytebase
is growing way faster.

- [ApiHug vs. Postman](/blog/apihug-vs-postman/)
- [ApiHug vs. Stoplight](/blog/apihug-vs-stoplight/)

### SQL GUI Client

SQL GUI Client such as MySQL Workbench, pgAdmin, DBeaver, Navicat provide a GUI to interact with the
database. ApiHug not only provides a GUI client, it can also enforce centralized data access control
for data security and governance.
