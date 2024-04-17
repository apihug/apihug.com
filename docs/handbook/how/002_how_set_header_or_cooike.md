---
title: 如何支持返回HTTP头或者Cookie
description: 返回http头，或者设置cookie
---

😆 视频教程：

1. [ApiHug101-Bilibili](https://space.bilibili.com/666522636)
2. [ApiHug101-Youtube](https://youtube.com/@ApiHug?si=C1yw0poHA01zbmyj)

佐以IDEA plugin 更好食用：

💝 IDEA 插件： [ApiHug - API design Copilot](https://plugins.jetbrains.com/plugin/23534-apihug--api-design-copilot)  &nbsp;&nbsp; [Plugin Handbook](../IDE/README.md)  &nbsp;&nbsp; [Plugin FAQ](../IDE/999_FAQ.md)

非常简单, 无需任何设置， 在您的 `Service`实现，设置返回对象 `body` `header` 就可以， super easy!

## 实现

```java
private static HttpCookie buildCookie(final String jwt){
  return
          ResponseCookie.from("Authorization",jwt)
                  .path("/")
                  .maxAge(Duration.ofDays(1))
                  .httpOnly(true).build();
}

@Override
public void login(
    final SimpleResultBuilder<LoginResponse> builder, final LoginRequest loginRequest) {
      //...
      builder.body().header(HttpHeaders.SET_COOKIE, buildCookie(jwt).toString());
      //....
}
```

这个例子，我们将在HTTP 返回里，设置 `cookie`。
