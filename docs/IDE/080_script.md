---
title: IDE Plugin 前置后置脚本
description: API 脚本管理
---

⚠️⚠️ After SDK `0.6.8-RELEASE`+  & IDEA Plugin `0.2.5`+

## API 脚本管理

1. `groovy` 语法
2. 自动完成
3. 前置 request 操作， 后置 response 处理

## 内置对象

**通用** 部分：

| 接口          | 说明        | 备注        |
|-------------|------------|-----------|
| `String getHeader(String key)`     | 获取 header       |request/response都支持|
| `List<String> getHeaders(String key)`     |   获取 header列表值     |request/response都支持|
| `List<String> getHeaderKeys()`     |    获取 header名列表    |request/response都支持|

### Request

| 接口          | 说明        | 备注        |
|-------------|------------|-----------|
| `String getUrl()`     |    获取当前URL    |-|
| `void setUrl(String url)`     |    覆盖URL    |-|
| `String getParameter(String name)`     |   获取参数名     |多个list只取第一个|
| `void setParameter(String name, String value)`     |    设置参数    |覆盖已有的|
| `void setParameters(Map<String, String> parameters)`     |   设置参数列表     |覆盖已有的|
| `void addParameter(String name, String value)`     |    添加参数    |没有新建，有加入|
| `void removeParameter(String name)`     |     删除参数   |-|
| `void setFormData(String name, String value)`     |    设置表单数据    |-|
| `void setHeader(String name, String value)`     |     设置header   |覆盖已有|
| `void addHeader(String name, String value)`     |     设置header   |添加到已有|
| `void removeHeader(String name)`     |    删除header    |-|
| `byte[] getRequestBody()`     |     获取请求body   |-|
| `void setRequestBody(String payload)`     |    设置请求body    |字符|
| `void setRequestBody(byte[] payload)`     |    设置请求body    |byte|
| `void setRequestBody(InputStream input)`     |  设置请求body      |字节流|
| `Map<String, List<String>> getParameterMap()`     | 获取所有参数       |-|
| `List<String> getParameterValues(String name)`     |  获取所有参数名      |-|
| `void setCookie(String cookie)`     |     设置cookie   |-|

### Response

| 接口          | 说明        | 备注        |
|-------------|------------|-----------|
| `String getResponseBody()`     |   获取返回体     |-|
| `int getCode()`     |    返回http code    |判断成功失败|
| `String getCookie(String name)`     |  获取cookie      |根据名字|
| `String getCookie()`     |    获取cookie    |整个cookie|

### Logger

返回信息到IDEA控制台。

| 接口          | 说明        | 备注        |
|-------------|------------|-----------|
| `void clear()`     |     清空   |-|
| `void warn(final String message)`     |   warn     |-|
| `void info(final String message)`     |   info     |-|
| `void debug(final String message)`     |   debug     |-|
| `void error(final String message)`     |   error     |-|

### Runtime context

可用于根据返回信息，更新当前运行环境参数或者变量。

| 接口          | 说明        | 备注        |
|-------------|------------|-----------|
| `void updateGlobalVariable(String name, Object value)`     |   覆盖当前服务全局变量     |-|
| `void updateGlobalParameter(String name, Object value)`     |  覆盖当前服务全局参数      |-|
| `void updateEnvVariable(String name, Object value)`     |     覆盖当前环境变量   |-|
| `void updateEnvParameter(String name, Object value)`     |    覆盖当前环境参数    |-|

## Sample Template

注意， pay attention: ⚠️

1. `dummy` 对象为辅助你设计阶段，代码自动完成，提交前请注释掉(运行前可框架替换，但是如果变形无法处理)
2. 项目需要整体 gradle build 下，否则可能上下文获取不到，这个是一次性工作
3. groovy 语法和java 差不多， 基本通用，而且更简洁， 结尾无需 `;`
4. 使用上下文的 `logger`输出信息， 勿用`System.out`；

### Prior Script

上下文提供：

1. `request`， 原始请求对象
2. `logger`， logger 输出

```groovy

import hope.common.script.HTTPRequest
import hope.common.script.LoggerAdapter

/************************************************************************
 * 1. Do not touch code below, it will be injected at runtime
 * 2. Fellow the groovy syntax, Java syntax almost can work
 ************************************************************************/

HTTPRequest request = HTTPRequest.DUMMY
LoggerAdapter logger = LoggerAdapter.DUMMY

/*************************************************************************/

// Place your logic here

headerKey = request.getHeader("key");

if (headerKey != null) {
    logger.warn("we has header value $headerKey")
} else {
    logger.warn("we has no header value set")
}

//request.setHeader("key","value");


```

### Post Script

上下文提供：

1. `response`， 原始返回体
2. `logger`， logger 输出
3. `runtime`， 运行时环境变量修改

```groovy

import hope.common.script.HTTPResponse
import hope.common.script.LoggerAdapter
import hope.common.script.RuntimeContext

/************************************************************************
 * 1. Do not touch code below, it will be injected at runtime
 * 2. Fellow the groovy syntax, Java syntax almost can work
 ************************************************************************/

HTTPResponse response = HTTPResponse.DUMMY
LoggerAdapter logger = LoggerAdapter.DUMMY
RuntimeContext runtime = RuntimeContext.DUMMY

/*************************************************************************/

// Place your logic here

response.getHeader("key");

```

## Reference

1. [ApiHug101-Bilibili](https://www.bilibili.com/video/BV1KK421k7J8/)
2. [ApiHug101-Youtube](https://youtube.com/@ApiHug?si=C1yw0poHA01zbmyj)
