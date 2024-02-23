# Milestone(一)

⚠️ 由于修改不再兼容老的风格迁移有风险， 包含基础包,IDE plugin

⚠️ This is an incompatible change, Both the SDK and IDE plugin not compatible with previous version.

## ApiHug [0.6.0-RELEASE] -  2024-02-22

### SDK Features

- `google.protobuf.BoolValue` -> `hope.common.BoolType`, less code, more straightforward. [3 kinds Boolean](../blog/2024Q1/20240222-3-kinds-boolean-issue.md)
- message field deprecated flag picker logic
- `authorization_struct`  --> `rbac` make it more human friendly: `RBAC rbac = 2;`
- support `hope.common.BoolType blank = 49;` for the `String` specific field
   - `empty` vs `blank`,  blank more for the string field, string field may not empty, but may be blank, as it include no qualify character, like space, tab, etc.
- bug fix

## ApiHug-IDE [0.2.0] -  2024-02-22

### IDE Features

1. new wire info panel, link to enum
2. auto refresh after edit
3. upgrade the AST parser
4. authority auto complete
5. bugfix

## Steps

**升级依赖包+IDE版本**：

**Upgrade SDK+IDE Version**：

## Project

1. `{PROJECT}/gradle/libs.versions.toml`
2. `apihug = "OLD_VERSION"` -> `0.6.0-RELEASE`+

## Plugin

1. download: [ApiHug - API design Copilot](https://plugins.jetbrains.com/plugin/23534-apihug--api-design-copilot)  `0.2.0`+
2. upgrade in IDE:

### Code

**1.`google.protobuf.BoolValue` -> `hope.mock.BoolRule`**

🏁 find all the broken like:

```proto
    empty: {
      value: false
    };
```

**Change TO**:

```proto
    empty: FALSE;
```

**2. `authorization_struct`  --> `rbac`**

🏁 find all the broken like:

```proto
       authorization_struct: {
          authorities: "PLATFORM_MEMBER_OPERATE";
          combinator: OR;
          predefined_role_checker: PLATFORM_MANAGER
        }
      }
```

**Change TO**:

```proto
       rbac: {
          authorities: "PLATFORM_MEMBER_OPERATE";
          combinator: OR;
          predefined_role_checker: PLATFORM_MANAGER
        }
      }
```

🥳🥳🥳 all done!
