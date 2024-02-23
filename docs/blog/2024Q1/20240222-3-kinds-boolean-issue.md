---
title: 3 kinds Boolean support in ApiHug framework
description: bool should has 2 flag, but 3 design ways really freak out.
---

## 3 kinds Boolean

**proto primitive**:

```proto3
//Define
bool flag =1;

//Assign
flag: true;

```

**proto wrapper**:

```proto3
//Define
google.protobuf.BoolValue flag = 1;

//Assign
flag: {
    value: true;
}

```

**ApiHug Enum**:

```proto3
//Define
hope.common.BoolType flag = 1;

//Assign
flag: TRUE;

```

## Why

| ApiHug          | pros         | cons        | Comment        |
|-------------|------------|-----------|-----------|
| Proto Primitive     | straightforward        | default always false | as no default or optional present since proto3|
| Proto Wrapper     | support not set       | syntax a bit complex  | when assign you need a message wrapper |
| ApiHug Enum     | straightforward       | introduce depend on ApiHug Lib  | only used in meta generation no affect on application layer |

## Rule

Since ApiHug Lib `0.6.0-RELEASE` and IDE Plugin version `0.2.0`，avoid the usage of of Proto's primitive & wrapper ways, use ApiHug Enum as mandatory.

Otherwise the IDE plugin can not parse the AST tree, any bool dependent flag can not be picked up.

## Refer

1. [Avoid Null Booleans in Java](https://medium.com/swlh/avoid-null-booleans-in-java-4a5cd9b23bca)
2. [Wikipedia: Boolean algebra](https://en.wikipedia.org/wiki/Boolean_algebra)
3. [wikipedia: Three-valued logic](https://en.wikipedia.org/wiki/Three-valued_logic)
