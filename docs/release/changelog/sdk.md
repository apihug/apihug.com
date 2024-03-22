<!-- Keep a Changelog guide -> https://keepachangelog.com -->

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