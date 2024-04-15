---
title: 如何支持文件上传
description: 文件上传定义，文件上传例子，文件上传如何写?
---

😆 视频教程：

1. [ApiHug101-Bilibili](https://space.bilibili.com/666522636)
2. [ApiHug101-Youtube](https://youtube.com/@ApiHug?si=C1yw0poHA01zbmyj)

佐以IDEA plugin 更好食用：

💝 IDEA 插件： [ApiHug - API design Copilot](https://plugins.jetbrains.com/plugin/23534-apihug--api-design-copilot)  &nbsp;&nbsp; [Plugin Handbook](../IDE/README.md)  &nbsp;&nbsp; [Plugin FAQ](../IDE/999_FAQ.md)

三步骤定义文件上传支持接口

## Proto Request

定义属性为 `bytes` 类型：

```proto
message UploadBookCoverToLocalRequest {

  option (hope.swagger.schema) = {
    json_schema: {
      description: "upload a cover of book to the local file system";
    };
  };

  int64 id = 1 [(hope.swagger.field) = {
    description: "id of this book";
    empty: FALSE;
  }];

  bytes file = 2 [(hope.swagger.field) = {
    description: "file of this book must be a image(jpg|png) etc";
    empty: FALSE;
  }];
}
```

## 接口定义

定义接口 `consumes: "multipart/form-data"`:

```proto
rpc UploadBookCoverToLocal (com.novel.book.proto.api.admin.request.UploadBookCoverToLocalRequest) returns (google.protobuf.Empty) {
    option (hope.swagger.operation) = {
      post: "/upload-book-cover";
      description: "Upload a book cover to local file system";
      priority: LOW;
      consumes: "multipart/form-data";
      authorization:{
        rbac:{
          predefined_role_checker: PLATFORM
        }
      }
    };
  }
```

## Wire 获得接口定义

```json
  "/admin/book/upload-book-cover" : {
      "post" : {
        "tags" : [ ],
        "summary" : "",
        "description" : "Upload a book cover to local file system",
        "operationId" : "AdminBookService#UploadBookCoverToLocal",
        "parameters" : [ ],
        "requestBody" : {
          "content" : {
            "multipart/form-data" : {
              "schema" : {
                "type" : "object",
                "$ref" : "#/components/schemas/UploadBookCoverToLocalRequest"
              }
            }
          }
        }
      }
    }
```

![post file](../../public/image/how/post_file.png)

对于 `Multipart` 类型对象， 在 Wire 端仅生成一个 `Generic` 对象：

```java
public class UploadBookCoverToLocalRequest<Multipart> {
  @hope.common.service.annotation.Multipart
  @NotNull
  @Schema(
      description = "file of this book must be a image(jpg|png) etc",
      requiredMode = Schema.RequiredMode.REQUIRED
  )
  protected Multipart file;
}
```

具体实现， 在 stub, application 端生成。

## Stub 获得接口实现

生成对应实现 `Multipart` 对象：

```java

import javax.annotation.Generated;
import org.springframework.web.multipart.MultipartFile;

@Generated("H.O.P.E. Infra Team")
public class UploadBookCoverToLocalRequestMultipart extends UploadBookCoverToLocalRequest<MultipartFile> {
}
```

接下来就和普通的操作文件接口一样去操作上传文件啦！

```java
/**
   *
   * Authorization:
   *
   * <ul>
   * 	<li>PredefinedRoleCheckerType: PLATFORM</li>
   * </ul>
   * @apiNote
   * 	<p>{@code /admin/book/upload-book-cover}
   * 	<p>{@code Upload a book cover to local file system}
   */
  default void uploadBookCoverToLocal(SimpleResultBuilder<String> builder,
      UploadBookCoverToLocalRequestMultipart uploadBookCoverToLocalRequestMultipart) {
    builder.notImplemented();
  }
  ```
