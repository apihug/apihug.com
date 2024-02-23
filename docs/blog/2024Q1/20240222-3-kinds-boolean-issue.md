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

## Rule

## Refer

1. [Avoid Null Booleans in Java](https://medium.com/swlh/avoid-null-booleans-in-java-4a5cd9b23bca)
2. [Wikipedia: Boolean algebra](https://en.wikipedia.org/wiki/Boolean_algebra)
3. [wikipedia: Three-valued logic](https://en.wikipedia.org/wiki/Three-valued_logic)
