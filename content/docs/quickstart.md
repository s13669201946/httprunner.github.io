---
title: "快速上手"
weight: 20
description: 10 分钟内快速上手 HttpRunner
---

HttpRunner 的首要核心目标就是「简单易用」，即使你是新用户，你也可以在 10 分钟之内快速上手 HttpRunner。

那我们计时开始吧！🚀

## 安装部署

`HttpRunner v4` 采用 Golang 开发，已针对主流操作系统预编译了二进制文件，只需在系统终端中执行一条命令即可完成安装部署。

```bash
$ bash -c "$(curl -ksSL https://httprunner.com/script/install.sh)"
```

安装成功后，你将获得一个 `hrp` 命令行工具，执行 `hrp -h` 即可查看到参数帮助说明。

```text
$ hrp -h

██╗  ██╗████████╗████████╗██████╗ ██████╗ ██╗   ██╗███╗   ██╗███╗   ██╗███████╗██████╗
██║  ██║╚══██╔══╝╚══██╔══╝██╔══██╗██╔══██╗██║   ██║████╗  ██║████╗  ██║██╔════╝██╔══██╗
███████║   ██║      ██║   ██████╔╝██████╔╝██║   ██║██╔██╗ ██║██╔██╗ ██║█████╗  ██████╔╝
██╔══██║   ██║      ██║   ██╔═══╝ ██╔══██╗██║   ██║██║╚██╗██║██║╚██╗██║██╔══╝  ██╔══██╗
██║  ██║   ██║      ██║   ██║     ██║  ██║╚██████╔╝██║ ╚████║██║ ╚████║███████╗██║  ██║
╚═╝  ╚═╝   ╚═╝      ╚═╝   ╚═╝     ╚═╝  ╚═╝ ╚═════╝ ╚═╝  ╚═══╝╚═╝  ╚═══╝╚══════╝╚═╝  ╚═╝

HttpRunner is an open source API testing tool that supports HTTP(S)/HTTP2/WebSocket/RPC
network protocols, covering API testing, performance testing and digital experience
monitoring (DEM) test types. Enjoy! ✨ 🚀 ✨

License: Apache-2.0
Website: https://httprunner.com
Github: https://github.com/httprunner/httprunner
Copyright 2021 debugtalk

Usage:
  hrp [command]

Available Commands:
  boom         run load test with boomer
  completion   generate the autocompletion script for the specified shell
  har2case     convert HAR to json/yaml testcase files
  help         Help about any command
  pytest       run API test with pytest
  run          run API test with go engine
  startproject create a scaffold project

Flags:
  -h, --help               help for hrp
      --log-json           set log to json format
  -l, --log-level string   set log level (default "INFO")
  -v, --version            version for hrp

Use "hrp [command] --help" for more information about a command.
```

HttpRunner 还支持多种安装部署方式，详见用户指南的[安装说明]部分。

## 脚手架创建项目

HttpRunner 支持使用脚手架创建示例项目。

执行 `hrp startproject` 命令，即可初始化指定名称的项目工程。

```bash
$ hrp startproject demo
10:39PM INF Set log to color console other than JSON format.
10:39PM ??? Set log level
10:39PM INF create new scaffold project pluginType=py projectName=demo
10:39PM INF create folder path=demo
10:39PM INF create folder path=demo/har
10:39PM INF create file path=demo/har/.keep
10:39PM INF create folder path=demo/testcases
10:39PM INF create folder path=demo/reports
10:39PM INF create file path=demo/reports/.keep
10:39PM INF create file path=demo/.gitignore
10:39PM INF create file path=demo/.env
10:39PM INF create file path=demo/testcases/demo_with_funplugin.json
10:39PM INF create file path=demo/testcases/demo_requests.yml
10:39PM INF create file path=demo/testcases/demo_ref_testcase.yml
10:39PM INF start to create hashicorp python plugin
10:39PM INF create file path=demo/debugtalk.py
10:39PM INF python3 venv is ready funppyVersion=0.4.2 venvDir=/Users/debugtalk/.hrp/venv
10:39PM INF create scaffold success projectName=demo
```

如下是项目工程的目录结构：

```text
$ tree demo -a
demo
├── .env
├── .gitignore
├── debugtalk.py
├── har
│   └── .keep
├── reports
│   └── .keep
└── testcases
    ├── demo_ref_testcase.yml
    ├── demo_requests.yml
    └── demo_with_funplugin.json

3 directories, 8 files
```

其中，testcases 文件夹中包含了多个示例测试用例。关于 HttpRunner 测试工程的目录结构说明，可查看用户指南的[测试工程目录结构]部分。

HttpRunner 创建脚手架项目时默认选择 Python 插件模式（兼容 v4 以前的版本），但同时还支持多种其它模式，可查看用户指南的[脚手架创建项目]部分了解更多详情。

## 初览测试用例

接下来我们以 `demo_requests.yml` 为例，初步预览下 HttpRunner 的测试用例结构。

```yaml
config:
    name: "request methods testcase with functions"
    variables:
        foo1: config_bar1
        foo2: config_bar2
        expect_foo1: config_bar1
        expect_foo2: config_bar2
    base_url: "https://postman-echo.com"
    verify: False
    export: ["foo3"]

teststeps:
-
    name: get with params
    variables:
        foo1: bar11
        foo2: bar21
        sum_v: "${sum_two(1, 2)}"
    request:
        method: GET
        url: /get
        params:
            foo1: $foo1
            foo2: $foo2
            sum_v: $sum_v
        headers:
            User-Agent: HttpRunner/${get_httprunner_version()}
    extract:
        foo3: "body.args.foo2"
    validate:
        - eq: ["status_code", 200]
        - eq: ["body.args.foo1", "bar11"]
        - eq: ["body.args.sum_v", "3"]
        - eq: ["body.args.foo2", "bar21"]
-
    name: post raw text
    variables:
        foo1: "bar12"
        foo3: "bar32"
    request:
        method: POST
        url: /post
        headers:
            User-Agent: HttpRunner/${get_httprunner_version()}
            Content-Type: "text/plain"
        data: "This is expected to be sent back as part of response body: $foo1-$foo2-$foo3."
    validate:
        - eq: ["status_code", 200]
        - eq: ["body.data", "This is expected to be sent back as part of response body: bar12-$expect_foo2-bar32."]
-
    name: post form data
    variables:
        foo2: bar23
    request:
        method: POST
        url: /post
        headers:
            User-Agent: HttpRunner/${get_httprunner_version()}
            Content-Type: "application/x-www-form-urlencoded"
        data: "foo1=$foo1&foo2=$foo2&foo3=$foo3"
    validate:
        - eq: ["status_code", 200]
        - eq: ["body.form.foo1", "$expect_foo1"]
        - eq: ["body.form.foo2", "bar23"]
        - eq: ["body.form.foo3", "bar21"]
```

HttpRunner 测试用例包括且仅有两部分：

- config：测试用例的公共配置部分，包括用例名称、base_url、参数化数据源、是否开启 SSL 校验等
- teststeps：有序步骤的集合；采用了 `go interface` 的设计理念，支持进行任意协议和测试类型的拓展（甚至包括 UI 自动化）

在上面的案例中，每个 step 都是一个 HTTP 请求；可以看到，描述信息仅包含了 HTTP 请求和结果校验的核心要素，没有任何累赘的内容。

同时，需要重点关注的是，虽然上面的用例是 YAML 文本，但同样支持引用变量和调用函数。

- 变量引用：约定通过 `${}` 或 `$` 的形式来引用变量，例如 `$foo1` 或 `${foo1}`
- 函数调用：约定通过 `${}` 的形式来调用插件函数，例如 `${sum_two(1, 2)}`

变量的申明定义在 step 或 config 的 `variables` 中，并且遵循优先级的要求。

函数的申明定义在项目根目录的 `debugtalk.py` 中，基于「约定大于配置」的设计理念，我们无需在测试用例中进行配置。

```python
import funppy


def get_httprunner_version():
    return "v4.0.0-alpha"


def sum_two_int(a: int, b: int) -> int:
    return a + b


if __name__ == '__main__':
    funppy.register("get_httprunner_version", get_httprunner_version)
    funppy.register("sum_two", sum_two_int)
    funppy.serve()
```

在 `debugtalk.py` 中，我们可以编写实现任意自定义逻辑的函数，只需通过 `funppy` 进行 `register` 和 `serve()` 即可。

关于变量、函数、插件等概念，可详细阅读[核心概念]。

## 运行接口测试

测试用例就绪后，通过 `hrp run` 命令即可执行指定的测试用例；如需生成 HTML 测试报告，可附带 `--gen-html-report` 参数。

<details>
<summary>$ hrp run testcases/demo_requests.yml testcases/demo_ref_testcase.yml --gen-html-report</summary>

```text
$ cd demo/
$ hrp run testcases/demo_requests.yml testcases/demo_ref_testcase.yml --gen-html-report
11:28PM INF Set log to color console other than JSON format.
11:28PM ??? Set log level
11:28PM INF [init] SetFailfast failfast=true
11:28PM INF [init] SetSaveTests saveTests=false
11:28PM INF [init] SetgenHTMLReport genHTMLReport=true
11:28PM INF [init] SetRequestsLogOn
11:28PM INF load file path=testcases/demo_requests.yml
11:28PM INF load testcases successfully count=1
11:28PM INF init session runner
11:28PM INF run testcase start testcase="request methods testcase with functions"
11:28PM INF init session runner
11:28PM INF python3 venv is ready funppyVersion=0.4.2 venvDir=/Users/debugtalk/.hrp/venv
11:28PM INF load hashicorp go plugin success path=/Users/debugtalk/MyProjects/HttpRunner-dev/httprunner.github.io/demo/debugtalk.py
11:28PM INF run step start step="get with params" type=request-GET
11:28PM INF function GetNames called on host side
11:28PM INF call function via gRPC funcArgs=[1,2] funcName=sum_two
11:28PM INF call function success arguments=[1,2] funcName=sum_two output=3
11:28PM INF function GetNames called on host side
11:28PM INF call function via gRPC funcArgs=[] funcName=get_httprunner_version
11:28PM INF call function success arguments=[] funcName=get_httprunner_version output=v4.0.0-alpha
-------------------- request --------------------
GET /get?foo1=bar11&foo2=bar21&sum_v=3 HTTP/1.1
Host: postman-echo.com
User-Agent: HttpRunner/v4.0.0-alpha


==================== response ===================
HTTP/1.1 200 OK
Content-Length: 335
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Sun, 10 Apr 2022 15:28:26 GMT
Etag: W/"14f-7Gzvv7hBEzQUyUYoNokC4etN280"
Set-Cookie: sails.sid=s%3ArupBebwxLVsoyG0nCfHMNjM-4u_aPEYs.tfgPzcR7kRyd2zuJqv7Vb%2FxlUJKFilYWoOtxXu7QsxA; Path=/; HttpOnly
Vary: Accept-Encoding

{"args":{"foo1":"bar11","foo2":"bar21","sum_v":"3"},"headers":{"x-forwarded-proto":"https","x-forwarded-port":"443","host":"postman-echo.com","x-amzn-trace-id":"Root=1-6252f79a-0e1cf4283dee310914d69da1","user-agent":"HttpRunner/v4.0.0-alpha","accept-encoding":"gzip"},"url":"https://postman-echo.com/get?foo1=bar11&foo2=bar21&sum_v=3"}
--------------------------------------------------
11:28PM INF extract value from=body.args.foo2 value=bar21
11:28PM INF set variable value=bar21 variable=foo3
11:28PM INF validate status_code assertMethod=eq checkExpr=status_code checkValue=200 expectValue=200 result=true
11:28PM INF validate body.args.foo1 assertMethod=eq checkExpr=body.args.foo1 checkValue=bar11 expectValue=bar11 result=true
11:28PM INF validate body.args.sum_v assertMethod=eq checkExpr=body.args.sum_v checkValue=3 expectValue=3 result=true
11:28PM INF validate body.args.foo2 assertMethod=eq checkExpr=body.args.foo2 checkValue=bar21 expectValue=bar21 result=true
11:28PM INF run step end exportVars={"foo3":"bar21"} step="get with params" success=true type=request
11:28PM INF run step start step="post raw text" type=request-POST
11:28PM INF call function via gRPC funcArgs=[] funcName=get_httprunner_version
11:28PM INF call function success arguments=[] funcName=get_httprunner_version output=v4.0.0-alpha
-------------------- request --------------------
POST /post HTTP/1.1
Host: postman-echo.com
Content-Type: text/plain
User-Agent: HttpRunner/v4.0.0-alpha

This is expected to be sent back as part of response body: bar12-config_bar2-bar32.
==================== response ===================
HTTP/1.1 200 OK
Content-Length: 441
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Sun, 10 Apr 2022 15:28:26 GMT
Etag: W/"1b9-biNClRDBTZ3yYGizMLRqBbpZaqc"
Set-Cookie: sails.sid=s%3AShiSgikIS1udNVe0IOdEVNf9rL_125st.D6%2Fw%2BIyKg5jeiOs6xwL8wsEht4tcO%2BpwXNqNWoFZQ3o; Path=/; HttpOnly
Vary: Accept-Encoding

{"args":{},"data":"This is expected to be sent back as part of response body: bar12-config_bar2-bar32.","files":{},"form":{},"headers":{"x-forwarded-proto":"https","x-forwarded-port":"443","host":"postman-echo.com","x-amzn-trace-id":"Root=1-6252f79a-75965ac66ae0074d0f1df200","content-length":"83","user-agent":"HttpRunner/v4.0.0-alpha","content-type":"text/plain","accept-encoding":"gzip"},"json":null,"url":"https://postman-echo.com/post"}
--------------------------------------------------
11:28PM INF validate status_code assertMethod=eq checkExpr=status_code checkValue=200 expectValue=200 result=true
11:28PM INF validate body.data assertMethod=eq checkExpr=body.data checkValue="This is expected to be sent back as part of response body: bar12-config_bar2-bar32." expectValue="This is expected to be sent back as part of response body: bar12-config_bar2-bar32." result=true
11:28PM INF run step end exportVars=null step="post raw text" success=true type=request
11:28PM INF run step start step="post form data" type=request-POST
11:28PM INF call function via gRPC funcArgs=[] funcName=get_httprunner_version
11:28PM INF call function success arguments=[] funcName=get_httprunner_version output=v4.0.0-alpha
-------------------- request --------------------
POST /post HTTP/1.1
Host: postman-echo.com
Content-Type: application/x-www-form-urlencoded
User-Agent: HttpRunner/v4.0.0-alpha

foo1=config_bar1&foo2=bar23&foo3=bar21
==================== response ===================
HTTP/1.1 200 OK
Content-Length: 479
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Sun, 10 Apr 2022 15:28:27 GMT
Etag: W/"1df-+YTOdiT8pYPlN+1k+z3s4LiSJJw"
Set-Cookie: sails.sid=s%3A5E7k9yIyZQq2qk-So2GS_h2eCMGeosyg.rmNVhBKq49FZm%2BXaH33KdzEqlPwwveKMO5rxh2%2F%2FvFo; Path=/; HttpOnly
Vary: Accept-Encoding

{"args":{},"data":"","files":{},"form":{"foo1":"config_bar1","foo2":"bar23","foo3":"bar21"},"headers":{"x-forwarded-proto":"https","x-forwarded-port":"443","host":"postman-echo.com","x-amzn-trace-id":"Root=1-6252f79b-3c6359e66dad26597ece3881","content-length":"38","user-agent":"HttpRunner/v4.0.0-alpha","content-type":"application/x-www-form-urlencoded","accept-encoding":"gzip"},"json":{"foo1":"config_bar1","foo2":"bar23","foo3":"bar21"},"url":"https://postman-echo.com/post"}
--------------------------------------------------
11:28PM INF validate status_code assertMethod=eq checkExpr=status_code checkValue=200 expectValue=200 result=true
11:28PM INF validate body.form.foo1 assertMethod=eq checkExpr=body.form.foo1 checkValue=config_bar1 expectValue=config_bar1 result=true
11:28PM INF validate body.form.foo2 assertMethod=eq checkExpr=body.form.foo2 checkValue=bar23 expectValue=bar23 result=true
11:28PM INF validate body.form.foo3 assertMethod=eq checkExpr=body.form.foo3 checkValue=bar21 expectValue=bar21 result=true
11:28PM INF run step end exportVars=null step="post form data" success=true type=request
11:28PM INF run testcase end testcase="request methods testcase with functions"
11:28PM INF quit hashicorp plugin process
```
</details>

测试生成的 HTML 报告如下所示。

<img src="/image/demo-html-report.png" alt="HttpRunner html report" width="600">

在 HTML 测试报告中，可以点击查看到详细的请求内容和响应结果，详见 [demo-html-report]。

## 运行性能测试

针对已有的接口测试用例，HttpRunner 无需任何额外的工作，即可通过 `hrp boom` 命令运行性能测试；通过 `--spawn-count` 参数可指定并发用户数，通过 `--spawn-rate` 可指定起始发压斜率。


<details>
<summary>$ hrp boom testcases/demo_requests.yml --spawn-count 100 --spawn-rate 10</summary>

</details>

## 总结

通过上述演示，相信大家已经对 HttpRunner 有了一个初步的印象，并且应该可以初步将 HttpRunner 上手跑起来了。

限于篇幅，本文仅展示了 HttpRunner 最基础的功能，大家可以通过进一步阅读[用户指南]探索 HttpRunner 的全部功能特性。



[安装说明]: /docs/user-guide/installation/
[测试工程目录结构]: /docs/user-guide/project-structure/
[脚手架创建项目]: /docs/user-guide/scaffold/
[核心概念]: /docs/introduction/concepts/
[demo-html-report]: /reports/demo-report.html
[用户指南]: /docs/user-guide/
