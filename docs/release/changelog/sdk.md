<!-- Keep a Changelog guide -> https://keepachangelog.com -->

## [0.8.5-RELEASE] - 2024-04-14

### Features

- Sort `EnumWriter`, `PojoWriter`, `ServiceWriter` to avoid code conflict for merger for `ApiCollector`
- Add sort of `getConvertersToRegister` & `basePackages` & `Database` avoid code chaos during merger!
- `Java parser` not support `17` syntax

## [0.7.8-RELEASE] - 2024-04-12

### Features

- Move the `Repository` maintain way to `Trait` separated location, make code clean;
- `lite` as default model for the stub side, make it more swift

## [0.7.5-RELEASE] - 2024-04-10

### Features

- Tiny adjust of the `SecurityCustomizer` code template make the comment style perfect
- Enhance `JWTFilter` make it support customized `jwt` pick for example from header or cookies etc.
- Service implement right package judgement

## [0.7.4-RELEASE] - 2024-03-22
### Features
- Tiny code template change: Combinator must has both authorities and roles, feign configuration comments

## [0.7.3-RELEASE] - 2024-03-15
### Features
- fix feign stub issue, code break

## [0.7.0-RELEASE] - 2024-03-15
### Features
- introduce lite model to make the stub task easier

## [0.6.9-RELEASE] - 2024-03-13
### Features
- add script definition

## [0.6.5-RELEASE] - 2024-03-08
### Features
- not `blank` & `empty`, consider as `required` implicitly
- `read_only` support
- `OAS` sort as alphabetically descending


## [0.6.3-RELEASE] - 2024-03-01
### Features
- Empty Stub module can stub
- extract the runtime swagger to cache with name patter: `final String name = groupId + "_" + artifactId + ".json";` avoid possible conflict

## [0.6.1-RELEASE] - 2024-02-29

### Features 
- fix `defaultValue = 1` of the `@Schema` annotation to  `defaultValue = "1"`

## [0.6.0-RELEASE] -  2024-02-25

### Features

- `google.protobuf.BoolValue` -> `hope.mock.BoolRule`, less code, more straightforward.
-  message field deprecated flag picker logic
-  bug fix

## [0.5.7-RELEASE] -  2024-02-22

### Features
- `authorization_struct`  --> `RBAC` make it more user friendly: `RBAC rbac = 2;`
-  support `google.protobuf.BoolValue blank = 49;` for the `String` specific field
   -  `empty` vs `blank`,  blank more for the string field, string field may not empty, but may be blank, as it include no qualify character, like space, tab, etc.