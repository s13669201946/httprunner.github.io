---
title: 使用 HttpRunner v4.0 实现 HTTP/2 的测试
slug: how-to-test-http2
date: 2022-05-12
---

## 前言回顾

此前的文章 [HttpRunner v4.0 正式发布] 中提到，HttpRunner v4.0 已不再局限于 HTTP 协议，而是会拓展支持更多种类的网络协议，截至当前，HttpRunner v4.0 中 go 引擎已经新增支持了 HTTP/2 协议 和 WebSocket 协议，python 引擎也已经新增支持了 SQL 操作和 Thrift RPC 协议。本期文章将对支持 HTTP/2 的新特性进行介绍，对于 HttpRunner v4.0 的其他新特性，之后也将会陆续进行介绍，敬请期待。

## 特性介绍

对于原有的包含 HTTP/1.1 请求的测试脚本，将协议类型升级为 HTTP/2 的修改成本非常低，在添加请求参数、请求头和请求体等操作方面，HTTP/2 的测试用例与 HTTP/1.1 的风格基本保持一致，同时也继承了 HttpRunner 强大的参数关联和结果断言等功能。下面介绍将测试用例中的协议类型由 HTTP/1.1 升级为 HTTP/2 的具体修改方式。

### JSON/YAML 形态的测试用例

对于 JSON/YAML 形态的测试用例，只需要在 `request` 对象中添加一个新字段 `http2` 并且指定为 `true`。另外，新版本 HttpRunner 也支持对协议类型的断言（"check": "proto"），这对于判断服务端是否支持 HTTP/2 的场景来说是非常实用的，因为有些时候服务端不支持 HTTP/2 并且会自动降级为 HTTP/1.1，这时就有必要对协议类型进行断言。如下是一个 HTTP 协议升级之后的 JSON 测试用例，在原来的基础上，只增加了一行代码 `"http2": true` 即可将 HTTP/1.1 升级为 HTTP/2，YAML 脚本同理，此处不再赘述。
```json
{
    "config": {
        "name": "run request with HTTP/2",
        "base_url": "https://postman-echo.com"
    },
    "teststeps": [
        {
            "name": "HTTP/2 get",
            "request": {
                "method": "GET",
                "url": "/get",
                "http2": true,
                "params": {
                    "foo1": "foo1",
                    "foo2": "foo2"
                },
                "headers": {
                    "User-Agent": "HttpRunnerPlus"
                }
            },
            "validate": [
                {
                    "check": "status_code",
                    "assert": "equals",
                    "expect": 200,
                    "msg": "check status code"
                },
                {
                    "check": "proto",
                    "assert": "equals",
                    "expect": "HTTP/2.0",
                    "msg": "check protocol type"
                },
                {
                    "check": "body.args.foo1",
                    "assert": "length_equals",
                    "expect": 4,
                    "msg": "check param foo1"
                }
            ]
        }
    ]
}
```

### go test 形态的测试用例

对于 go test 形态的测试用例，将 HTTP/1.1 升级为 HTTP/2 的方式同样非常简单，只需要在调用链中的 GET、POST 等请求方法之前先调用 HTTP2() 即可。如下是一个 HTTP 协议升级之后的 go test 测试用例，请注意 hrp.NewStep 方法与 GET、POST 请求方法之间新增的 HTTP2() 调用。
```go
package tests

import (
   "testing"

   "github.com/httprunner/httprunner/hrp"
)

func TestHTTPProtocol(t *testing.T) {
   testcase := &hrp.TestCase{
      Config: hrp.NewConfig("run request with HTTP/1.1 and HTTP/2").
         SetBaseURL("https://postman-echo.com"),
      TestSteps: []hrp.IStep{
         hrp.NewStep("HTTP/2 get").
            HTTP2().
            GET("/get").
            WithParams(map[string]interface{}{"foo1": "foo1", "foo2": "foo2"}).
            WithHeaders(map[string]string{"User-Agent": "HttpRunnerPlus"}).
            Validate().
            AssertEqual("status_code", 200, "check status code").
            AssertEqual("proto", "HTTP/2.0", "check protocol type").
            AssertLengthEqual("body.args.foo1", 4, "check param foo1"),
         hrp.NewStep("HTTP/2 post").
            HTTP2().
            POST("/post").
            WithHeaders(map[string]string{"User-Agent": "HttpRunnerPlus"}).
            WithBody(map[string]interface{}{"foo1": "foo1", "foo2": "foo2"}).
            Validate().
            AssertEqual("status_code", 200, "check status code").
            AssertEqual("proto", "HTTP/2.0", "check protocol type").
            AssertLengthEqual("body.json.foo1", 4, "check body foo1"),
      },
   }
   err := hrp.NewRunner(t).Run(testcase)
   if err != nil {
      t.Fatalf("run testcase error: %v", err)
   }
}
```

## 注意事项

- 在使用 `hrp run` 进行接口测试时，通过命令行选项 `--proxy-url` 设置的代理 URL 对于 HTTP/2 协议的请求不会生效，原因是底层依赖不支持设置代理 URL
- 在使用 `hrp boom` 进行性能测试时，通过命令行选项 `--disable-keepalive` 设置的打开/关闭长连接对于 HTTP/2 协议的请求不会生效，原因是底层依赖不支持设置改参数，另外 HTTP/2 协议支持多路复用，可以天然地达到类似长连接的效果
- 虽然 RFC 文档中没有强制规定，不过目前大部分 HTTP/2 服务是建立在 SSL/TLS 之上的，因此建议在 URL 中使用 HTTPS

## What's next

目前 `hrp har2case` 命令还不支持将包含 HTTP/2 请求的 har 包转换为 JSON/YAML 测试用例，HttpRunner v4.0 之后的版本中将会对该特性进行支持

> 本文作者：卜卜星（HttpRunner 核心开发者）<br/>
> HttpRunner 项目官网: https://httprunner.com/<br/>
> 如果 HttpRunner 对你有过帮助，麻烦帮忙给个 ⭐️star⭐️ 鼓励下吧<br/>
> https://github.com/httprunner/httprunner

[HttpRunner v4.0 正式发布]: /blog/release-v4/
