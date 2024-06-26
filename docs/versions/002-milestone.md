# Milestone(二)

This milestone for the eventually SDK 1.0.0 release preparation;

Since `2024-05-10`, this upgrade may introduce code broken!⚠️

## Rename  `cn_message` to `message2`

Remove the support for  `cn_message` use the `message2` instead for the constant option;

```protobuf
message Meta {

  //this is the code of this constant, without the offset
  //the framework will hand this stuff
  int32 code = 1;

  // this is the description of this constant
  // this should be primary language supported(for example english)
  string message = 2;

  // Deprecated since 2024-05-08 from logic
  // Then will be removed before 1.0.0 official version
  // eventually use the i18n to solve this issue
  string cn_message = 3 [deprecated = true];

  // this is another description of this constant
  // This will be the second language supported(for example chinese)
  string message2 = 4;

  // those for the error extension
  // if this set then this will
  Error error = 5;
}
```

### Migration steps

#### Prepare

<a target="_blank" href="https://search.maven.org/artifact/com.apihug/it-bom"><img src="https://img.shields.io/maven-central/v/com.apihug/it-bom.svg?label=Maven%20Central" /></a>

[ApiHug - API design Copilot](https://plugins.jetbrains.com/plugin/23534-apihug--api-design-copilot)

1. upgrade the IDEA plugin to `0.4.0`+
2. upgrade the SDK to `0.9.8-RELEASE`+
   1. open file: `{PROJECT}/gradle/libs.versions.toml`
   2. find and update: `apihug = "OLD_VERSION"` -> `0.9.8-RELEASE`+

#### Code change

1. reload the gradle
2. exist-enum wire class will report error `descriptionZhCN` overwrite illegal;
3. try to find all the `setDescriptionZhCN` of the enum replace them with `setDescription2`; (for error description upgrade)
4. try to find all the `descriptionZhCN` of the enum replace them with `description2`; (for enum description upgrade)
5. change all the `cn_message` in `proto` to  `message2`, as the `cn_message` will be removed soon.

after all the error gone,  then rebuild, everything goes well again!

## New Feature

### Support Entity Column Name Customization

in the `hope-wire.json` define:

```json
{
  "persistence": {
    "identifyType": "LONG",
    "tenantType": "LONG",
    "format": "CAMEL",
    "upper": "UPPER"
  }
}
```

1. Format
   1. `DEFAULT`: as old framework, convert to `SNAKE` style
   2. `CAMEL`: `myVariableName`
   3. `SNAKE`: `my_variable_name`
2. Upper
   1. `DEFAULT`:as old framework, convert to `UPPER` style
   2. `UPPER`: `name` -> `NAME`
   3. `LOWER`: `NAME` -> `name`
   4. `CAPITALIZE`:  `userName` -> `UserName`
3. Exception: if you set column name manually in the proto already, this will always be the highest priority!
4. `hope.common.persistence.plugin.NameMappingStrategy` client plugin to rename the column name.

case **3**, the column will always be `MY_USER_NAME` as manually set as: `name: "MY_USER_NAME"`;

```protobuf
  string user_name = 1 [(hope.persistence.column) = {
    name: "MY_USER_NAME",
    description: "name of the account",
    nullable: FALSE,
    updatable: FALSE,
    length: {
      value: 32
    },
    type: VARCHAR
  }];
  ```

### Support Only Proto Build Strategy

Latest Plugin(since 0.4.0), project template will introduce build proto modules only flag.

But if your project already exist please follow this to update to support, as this only some gradle tricky.

open: `{PROJECT}/settings.gradle`:

1. replace `YOUR_PROTO_MODULE` with your proto module;
2. replace `YOUR_APP_MODULE` with your app module;

```groovy
def onlyProto = System.getProperty("onlyProto")?.toString();

if(onlyProto ==null){
    onlyProto = System.getenv('onlyProto')?.toString();
}

// If onlyProto property is not set or is not a valid boolean, default to false
// `-DonlyProto=true` to enable this
onlyProto = onlyProto != null ? onlyProto.toBoolean() : false

// Include
include('YOUR_PROTO_MODULE')

if(!onlyProto) {
    include('YOUR_APP_MODULE')
}else{
    logger.quiet "-DonlyProto=true so no app modules will be included"
}

```

Then you can use this shell:  `gradlew clean build -x test -x wireTest -DonlyProto=true`，

to build your proto modules quickly, especially when you has broken update in the proto!
