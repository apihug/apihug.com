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

### Prior Script

```groovy

import hope.common.script.HTTPRequest
import hope.common.script.LoggerAdapter

/**
 * 1. Comment below two lines of code before you submit this script
 * 2. Un-comment it for easy code auto-complete during edit mode
 * 3. Fellow the groovy syntax, Java syntax almost can work
 */


HTTPRequest request = HTTPRequest.DUMMY
LoggerAdapter logger= LoggerAdapter.DUMMY

//request.getHeader("key");
//request.setHeader("key","value");


```

### Post Script

```groovy

import hope.common.script.HTTPResponse
import hope.common.script.LoggerAdapter
import hope.common.script.RuntimeContext

/**
 * 1. Comment below two lines of code before you submit this script
 * 2. Un-comment it for easy code auto-complete during edit mode
 * 3. Fellow the groovy syntax, Java syntax almost can work
 */

HTTPResponse response = HTTPResponse.DUMMY
LoggerAdapter logger= LoggerAdapter.DUMMY
RuntimeContext runtime = RuntimeContext.DUMMY

response.getHeader("key");

```

## Reference

1. [ApiHug101-Bilibili](https://www.bilibili.com/video/BV1KK421k7J8/)
2. [ApiHug101-Youtube](https://youtube.com/@ApiHug?si=C1yw0poHA01zbmyj)
