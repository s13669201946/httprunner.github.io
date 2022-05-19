---
title: 接口耗时分析
weight: 9
description: 使用 HttpRunner 实现 HTTP(S) 接口耗时分析
---

在对 HTTP 接口做测试时，大多数时候我们只需要关注接口的整体耗时。那如果测试发现接口整体耗时较长，我们是否可以更进一步地分析原因呢？

实际上，单次 HTTP(S) 请求是会包含多个步骤的，依次包括 DNS 解析、TCP 连接、TLS 握手、服务端处理、数据传输 5 个环节；如果是 HTTP 协议，由于少了 TLS 握手环节，因此会包含 4 个环节。

HttpRunner 从 v4.0 版本开始，新增支持了 HTTP 接口耗时详情的采集能力，我们可以通过该能力实现对接口整体耗时的诊断分析。

## 采集耗时详情

默认情况下，通过 `hrp run` 执行接口测试时，接口耗时详情信息是不会采集的。如果要开启，只需要增加一个参数 `--http-stat` 即可。

示例如下：

```bash
$ hrp run demo/testcases/demo_requests.yml --http-stat
```

<details>
<summary>查看运行日志</summary>

```text
$ hrp startproject demo
$ hrp run demo/testcases/demo_requests.yml --http-stat
4:40PM INF Set log to color console other than JSON format.
4:40PM ??? Set log level
4:40PM INF [init] SetFailfast failfast=true
4:40PM INF [init] SetSaveTests saveTests=false
4:40PM INF [init] SetRequestsLogOn
4:40PM INF [init] SetHTTPStatOn
4:40PM INF load file path=demo/testcases/demo_requests.yml
4:40PM INF load testcases successfully count=1
4:40PM INF ensure python3 venv packages=["funppy"] python3=/Users/debugtalk/.hrp/venv/bin/python3
4:40PM INF python package is ready name=funppy version=0.4.3
4:40PM INF load hashicorp go plugin success path=/Users/debugtalk/MyProjects/HttpRunner-dev/httprunner/demo/debugtalk.py
4:40PM INF reset session runner
4:40PM INF run testcase start testcase="request methods testcase with functions"
4:40PM INF reset session runner
4:40PM INF run step start step="get with params" type=request-GET
4:40PM INF function GetNames called on host side
4:40PM INF call function via gRPC funcArgs=[1,2] funcName=sum_two_int
4:40PM INF call function success arguments=[1,2] funcName=sum_two_int output=3
4:40PM INF function GetNames called on host side
4:40PM INF call function via gRPC funcArgs=[] funcName=get_version
4:40PM INF call function success arguments=[] funcName=get_version output=0.4.3
-------------------- request --------------------
GET /get?foo1=bar11&foo2=bar21&sum_v=3 HTTP/1.1
Host: postman-echo.com
User-Agent: funplugin/0.4.3


==================== response ====================
Connected via TLSv1.2
HTTP/1.1 200 OK
Content-Length: 327
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Wed, 18 May 2022 08:40:45 GMT
Etag: W/"147-DFCO30rOwPI+4rpOE3phvU5pr+g"
Set-Cookie: sails.sid=s%3AkBCJ7WYvEkqRbTv2pC-gpM0rJQAYQ-6k.xYX5y2I%2B2pLRBkYF%2BBcfbo8V%2F8kMyTJM2ZxqqBSn1MM; Path=/; HttpOnly
Vary: Accept-Encoding

{"args":{"foo1":"bar11","foo2":"bar21","sum_v":"3"},"headers":{"x-forwarded-proto":"https","x-forwarded-port":"443","host":"postman-echo.com","x-amzn-trace-id":"Root=1-6284b10d-2636c23a45cb846420f8983f","user-agent":"funplugin/0.4.3","accept-encoding":"gzip"},"url":"https://postman-echo.com/get?foo1=bar11&foo2=bar21&sum_v=3"}
--------------------------------------------------

Connected to tcp: 34.205.194.84:443

  DNS Lookup   TCP Connection   TLS Handshake   Server Processing   Content Transfer
[     40ms  |         284ms  |        579ms  |            359ms  |             0ms  ]
            |                |               |                   |                  |
   namelookup:40ms           |               |                   |                  |
                       connect:325ms         |                   |                  |
                                   pretransfer:905ms             |                  |
                                                     starttransfer:1264ms           |
                                                                                total:1264ms

4:40PM INF HTTP latency statistics httpstat(ms)={"Connect":325,"ContentTransfer":0,"DNSLookup":40,"NameLookup":40,"Pretransfer":905,"ServerProcessing":359,"StartTransfer":1264,"TCPConnection":284,"TLSHandshake":579,"Total":1264}
4:40PM INF extract value from=body.args.foo2 value=bar21
4:40PM INF set variable value=bar21 variable=foo3
4:40PM INF validate status_code assertMethod=eq checkExpr=status_code checkValue=200 checkValueType=int64 expectValue=200 expectValueType=int64 result=true
4:40PM INF validate body.args.foo1 assertMethod=eq checkExpr=body.args.foo1 checkValue=bar11 checkValueType=string expectValue=bar11 expectValueType=string result=true
4:40PM INF validate body.args.sum_v assertMethod=eq checkExpr=body.args.sum_v checkValue=3 checkValueType=string expectValue=3 expectValueType=string result=true
4:40PM INF validate body.args.foo2 assertMethod=eq checkExpr=body.args.foo2 checkValue=bar21 checkValueType=string expectValue=bar21 expectValueType=string result=true
4:40PM INF run step end exportVars={"foo3":"bar21"} step="get with params" success=true type=request
4:40PM INF run step start step="post raw text" type=request-POST
4:40PM INF call function via gRPC funcArgs=[] funcName=get_version
4:40PM INF call function success arguments=[] funcName=get_version output=0.4.3
-------------------- request --------------------
POST /post HTTP/1.1
Host: postman-echo.com
Content-Type: text/plain
User-Agent: funplugin/0.4.3

This is expected to be sent back as part of response body: bar12-config_bar2-bar32.
==================== response ====================
Connected via TLSv1.2
HTTP/1.1 200 OK
Content-Length: 433
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Wed, 18 May 2022 08:40:46 GMT
Etag: W/"1b1-5kSLP1LVtSRaFElEIXrDWLjtj/Q"
Set-Cookie: sails.sid=s%3A977L0stmdaA5kpweNYd-7xxMItFdhEDX.XYOoDmmav6hi1OJn6L1SAAlKg%2B8Y2XRnx4rShvgGoLc; Path=/; HttpOnly
Vary: Accept-Encoding

{"args":{},"data":"This is expected to be sent back as part of response body: bar12-config_bar2-bar32.","files":{},"form":{},"headers":{"x-forwarded-proto":"https","x-forwarded-port":"443","host":"postman-echo.com","x-amzn-trace-id":"Root=1-6284b10e-556c7d5c1aebb6d87ef2715b","content-length":"83","user-agent":"funplugin/0.4.3","content-type":"text/plain","accept-encoding":"gzip"},"json":null,"url":"https://postman-echo.com/post"}
--------------------------------------------------

  DNS Lookup   TCP Connection   TLS Handshake   Server Processing   Content Transfer
[      0ms  |           0ms  |          0ms  |            296ms  |             0ms  ]
            |                |               |                   |                  |
   namelookup:0ms            |               |                   |                  |
                       connect:0ms           |                   |                  |
                                   pretransfer:0ms               |                  |
                                                     starttransfer:296ms            |
                                                                                total:296ms

4:40PM INF HTTP latency statistics httpstat(ms)={"Connect":0,"ContentTransfer":0,"DNSLookup":0,"NameLookup":0,"Pretransfer":0,"ServerProcessing":296,"StartTransfer":296,"TCPConnection":0,"TLSHandshake":0,"Total":296}
4:40PM INF validate status_code assertMethod=eq checkExpr=status_code checkValue=200 checkValueType=int64 expectValue=200 expectValueType=int64 result=true
4:40PM INF validate body.data assertMethod=eq checkExpr=body.data checkValue="This is expected to be sent back as part of response body: bar12-config_bar2-bar32." checkValueType=string expectValue="This is expected to be sent back as part of response body: bar12-config_bar2-bar32." expectValueType=string result=true
4:40PM INF run step end exportVars=null step="post raw text" success=true type=request
4:40PM INF run step start step="post form data" type=request-POST
4:40PM INF call function via gRPC funcArgs=[] funcName=get_version
4:40PM INF call function success arguments=[] funcName=get_version output=0.4.3
-------------------- request --------------------
POST /post HTTP/1.1
Host: postman-echo.com
Content-Type: application/x-www-form-urlencoded
User-Agent: funplugin/0.4.3

foo1=config_bar1&foo2=bar23&foo3=bar21
==================== response ====================
Connected via TLSv1.2
HTTP/1.1 200 OK
Content-Length: 471
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Wed, 18 May 2022 08:40:46 GMT
Etag: W/"1d7-Q1SNpWmnmggpXlXx7yOS+xlhpp8"
Set-Cookie: sails.sid=s%3AFc8Du4G-VTCrgjl3A-rqp5_bP8dns2OQ.bSxaAM07Nl%2FIg7mLzqFYzhGy1EtlaXrj4Y3d9aNQ%2FJM; Path=/; HttpOnly
Vary: Accept-Encoding

{"args":{},"data":"","files":{},"form":{"foo1":"config_bar1","foo2":"bar23","foo3":"bar21"},"headers":{"x-forwarded-proto":"https","x-forwarded-port":"443","host":"postman-echo.com","x-amzn-trace-id":"Root=1-6284b10e-3c2749a8025fdf4d3eb177b2","content-length":"38","user-agent":"funplugin/0.4.3","content-type":"application/x-www-form-urlencoded","accept-encoding":"gzip"},"json":{"foo1":"config_bar1","foo2":"bar23","foo3":"bar21"},"url":"https://postman-echo.com/post"}
--------------------------------------------------

  DNS Lookup   TCP Connection   TLS Handshake   Server Processing   Content Transfer
[      0ms  |           0ms  |          0ms  |            346ms  |             3ms  ]
            |                |               |                   |                  |
   namelookup:0ms            |               |                   |                  |
                       connect:0ms           |                   |                  |
                                   pretransfer:0ms               |                  |
                                                     starttransfer:346ms            |
                                                                                total:349ms

4:40PM INF HTTP latency statistics httpstat(ms)={"Connect":0,"ContentTransfer":3,"DNSLookup":0,"NameLookup":0,"Pretransfer":0,"ServerProcessing":346,"StartTransfer":346,"TCPConnection":0,"TLSHandshake":0,"Total":349}
4:40PM INF validate status_code assertMethod=eq checkExpr=status_code checkValue=200 checkValueType=int64 expectValue=200 expectValueType=int64 result=true
4:40PM INF validate body.form.foo1 assertMethod=eq checkExpr=body.form.foo1 checkValue=config_bar1 checkValueType=string expectValue=config_bar1 expectValueType=string result=true
4:40PM INF validate body.form.foo2 assertMethod=eq checkExpr=body.form.foo2 checkValue=bar23 checkValueType=string expectValue=bar23 expectValueType=string result=true
4:40PM INF validate body.form.foo3 assertMethod=eq checkExpr=body.form.foo3 checkValue=bar21 checkValueType=string expectValue=bar21 expectValueType=string result=true
4:40PM INF run step end exportVars=null step="post form data" success=true type=request
4:40PM INF run testcase end testcase="request methods testcase with functions"
4:40PM INF quit hashicorp plugin process
```
</details>

## 查看耗时详情

在上面的运行日志中，可以在接口的响应中看到如下信息：

- 当次 HTTP 请求连接的服务器 `IP:Port`
- 当次 HTTP 请求的详细耗时

![](/image/httpstat.png)

## 生成 summary 文件

如果我们还想将 HTTP 接口耗时详情数据保存为结构化的数据，那我们可以再额外指定 `-s` 或 `--save-tests` 参数，即可存储到 `summary.json` 文件。

```bash
$ hrp run demo/testcases/demo_requests.yml --http-stat --save-tests
```

在生成的 `summary.json` 文件中，会新增一个 `httpstat` 字段。

```json
"httpstat": {
    "Connect": 313,
    "ContentTransfer": 0,
    "DNSLookup": 10,
    "NameLookup": 10,
    "Pretransfer": 868,
    "ServerProcessing": 487,
    "StartTransfer": 1355,
    "TCPConnection": 302,
    "TLSHandshake": 554,
    "Total": 1356
},
```

<details>
<summary>查看完整 summary.json</summary>

```json
{
    "success": true,
    "stat": {
        "testcases": {
            "total": 1,
            "success": 1,
            "fail": 0
        },
        "teststeps": {
            "total": 3,
            "successes": 3,
            "failures": 0
        }
    },
    "time": {
        "start_at": "2022-05-18T17:48:11.302224+08:00",
        "duration": 3.017276084
    },
    "platform": {
        "httprunner_version": "v4.1.0-alpha",
        "go_version": "go1.16.3",
        "platform": "darwin-amd64"
    },
    "details": [
        {
            "name": "request methods testcase with functions",
            "success": true,
            "stat": {
                "total": 3,
                "successes": 3,
                "failures": 0
            },
            "time": {
                "start_at": "2022-05-18T17:48:12.393419+08:00",
                "duration": 1.9261071140000001
            },
            "in_out": {
                "config_vars": {
                    "expect_foo1": "config_bar1",
                    "expect_foo2": "config_bar2",
                    "foo1": "config_bar1",
                    "foo2": "config_bar2"
                },
                "export_vars": {
                    "foo3": "bar21"
                }
            },
            "records": [
                {
                    "name": "get with params",
                    "step_type": "request",
                    "success": true,
                    "elapsed_ms": 1356,
                    "httpstat": {
                        "Connect": 313,
                        "ContentTransfer": 0,
                        "DNSLookup": 10,
                        "NameLookup": 10,
                        "Pretransfer": 868,
                        "ServerProcessing": 487,
                        "StartTransfer": 1355,
                        "TCPConnection": 302,
                        "TLSHandshake": 554,
                        "Total": 1356
                    },
                    "data": {
                        "success": true,
                        "req_resps": {
                            "request": {
                                "headers": {
                                    "User-Agent": "funplugin/0.4.3"
                                },
                                "method": "GET",
                                "params": {
                                    "foo1": "bar11",
                                    "foo2": "bar21",
                                    "sum_v": 3
                                },
                                "url": "/get"
                            },
                            "response": {
                                "body": "{\n    \"args\": {\n    \"foo1\": \"bar11\",\n    \"foo2\": \"bar21\",\n    \"sum_v\": \"3\"\n},\n    \"headers\": {\n    \"accept-encoding\": \"gzip\",\n    \"host\": \"postman-echo.com\",\n    \"user-agent\": \"funplugin/0.4.3\",\n    \"x-amzn-trace-id\": \"Root=1-6284c0dd-5979be0b353e154c3c196ef6\",\n    \"x-forwarded-port\": \"443\",\n    \"x-forwarded-proto\": \"https\"\n},\n    \"url\": \"https://postman-echo.com/get?foo1=bar11\\u0026foo2=bar21\\u0026sum_v=3\"\n}",
                                "cookies": {
                                    "sails.sid": "s%3APe-sKLKTurWDcSqQs5K1xq_k0f2yjA04.ow4l8OLzWTQrhFZ0XqvFjvhiFNW2CdJitQs6w9Xdf9g"
                                },
                                "headers": {
                                    "Connection": "keep-alive",
                                    "Content-Length": "327",
                                    "Content-Type": "application/json; charset=utf-8",
                                    "Date": "Wed, 18 May 2022 09:48:13 GMT",
                                    "Etag": "W/\"147-FN1NxFZ5B4Vt2onH1Pi1CRYRsMw\"",
                                    "Set-Cookie": "sails.sid=s%3APe-sKLKTurWDcSqQs5K1xq_k0f2yjA04.ow4l8OLzWTQrhFZ0XqvFjvhiFNW2CdJitQs6w9Xdf9g; Path=/; HttpOnly",
                                    "Vary": "Accept-Encoding"
                                },
                                "proto": "HTTP/1.1",
                                "status_code": 200
                            }
                        },
                        "validators": [
                            {
                                "check": "status_code",
                                "assert": "eq",
                                "expect": 200,
                                "check_value": 200,
                                "check_result": "pass"
                            },
                            {
                                "check": "body.args.foo1",
                                "assert": "eq",
                                "expect": "bar11",
                                "check_value": "bar11",
                                "check_result": "pass"
                            },
                            {
                                "check": "body.args.sum_v",
                                "assert": "eq",
                                "expect": "3",
                                "check_value": "3",
                                "check_result": "pass"
                            },
                            {
                                "check": "body.args.foo2",
                                "assert": "eq",
                                "expect": "bar21",
                                "check_value": "bar21",
                                "check_result": "pass"
                            }
                        ]
                    },
                    "content_size": 327,
                    "export_vars": {
                        "foo3": "bar21"
                    }
                },
                {
                    "name": "post raw text",
                    "step_type": "request",
                    "success": true,
                    "elapsed_ms": 279,
                    "httpstat": {
                        "Connect": 0,
                        "ContentTransfer": 0,
                        "DNSLookup": 0,
                        "NameLookup": 0,
                        "Pretransfer": 0,
                        "ServerProcessing": 278,
                        "StartTransfer": 278,
                        "TCPConnection": 0,
                        "TLSHandshake": 0,
                        "Total": 279
                    },
                    "data": {
                        "success": true,
                        "req_resps": {
                            "request": {
                                "body": "This is expected to be sent back as part of response body: bar12-config_bar2-bar32.",
                                "data": "This is expected to be sent back as part of response body: $foo1-$foo2-$foo3.",
                                "headers": {
                                    "Content-Type": "text/plain",
                                    "User-Agent": "funplugin/0.4.3"
                                },
                                "method": "POST",
                                "url": "/post"
                            },
                            "response": {
                                "body": "{\n    \"args\": {\n    \n},\n    \"data\": \"This is expected to be sent back as part of response body: bar12-config_bar2-bar32.\",\n    \"files\": {\n    \n},\n    \"form\": {\n    \n},\n    \"headers\": {\n    \"accept-encoding\": \"gzip\",\n    \"content-length\": \"83\",\n    \"content-type\": \"text/plain\",\n    \"host\": \"postman-echo.com\",\n    \"user-agent\": \"funplugin/0.4.3\",\n    \"x-amzn-trace-id\": \"Root=1-6284c0dd-1b706c817978b13925ac792f\",\n    \"x-forwarded-port\": \"443\",\n    \"x-forwarded-proto\": \"https\"\n},\n    \"json\": null,\n    \"url\": \"https://postman-echo.com/post\"\n}",
                                "cookies": {
                                    "sails.sid": "s%3AoGlPJKhkyFGgMNO6dRTrnUQXkOFcDV7J.rWIbyDkCGdPUTVbwUfY3FOXaM2cpB2UqTD3WeJoysYM"
                                },
                                "headers": {
                                    "Connection": "keep-alive",
                                    "Content-Length": "433",
                                    "Content-Type": "application/json; charset=utf-8",
                                    "Date": "Wed, 18 May 2022 09:48:13 GMT",
                                    "Etag": "W/\"1b1-WQ+Yr/smdfmnFbXNXSAqoNA3NEE\"",
                                    "Set-Cookie": "sails.sid=s%3AoGlPJKhkyFGgMNO6dRTrnUQXkOFcDV7J.rWIbyDkCGdPUTVbwUfY3FOXaM2cpB2UqTD3WeJoysYM; Path=/; HttpOnly",
                                    "Vary": "Accept-Encoding"
                                },
                                "proto": "HTTP/1.1",
                                "status_code": 200
                            }
                        },
                        "validators": [
                            {
                                "check": "status_code",
                                "assert": "eq",
                                "expect": 200,
                                "check_value": 200,
                                "check_result": "pass"
                            },
                            {
                                "check": "body.data",
                                "assert": "eq",
                                "expect": "This is expected to be sent back as part of response body: bar12-config_bar2-bar32.",
                                "check_value": "This is expected to be sent back as part of response body: bar12-config_bar2-bar32.",
                                "check_result": "pass"
                            }
                        ]
                    },
                    "content_size": 433
                },
                {
                    "name": "post form data",
                    "step_type": "request",
                    "success": true,
                    "elapsed_ms": 281,
                    "httpstat": {
                        "Connect": 0,
                        "ContentTransfer": 2,
                        "DNSLookup": 0,
                        "NameLookup": 0,
                        "Pretransfer": 0,
                        "ServerProcessing": 278,
                        "StartTransfer": 278,
                        "TCPConnection": 0,
                        "TLSHandshake": 0,
                        "Total": 281
                    },
                    "data": {
                        "success": true,
                        "req_resps": {
                            "request": {
                                "body": "foo1=config_bar1&foo2=bar23&foo3=bar21",
                                "data": "foo1=$foo1&foo2=$foo2&foo3=$foo3",
                                "headers": {
                                    "Content-Type": "application/x-www-form-urlencoded",
                                    "User-Agent": "funplugin/0.4.3"
                                },
                                "method": "POST",
                                "url": "/post"
                            },
                            "response": {
                                "body": "{\n    \"args\": {\n    \n},\n    \"data\": \"\",\n    \"files\": {\n    \n},\n    \"form\": {\n    \"foo1\": \"config_bar1\",\n    \"foo2\": \"bar23\",\n    \"foo3\": \"bar21\"\n},\n    \"headers\": {\n    \"accept-encoding\": \"gzip\",\n    \"content-length\": \"38\",\n    \"content-type\": \"application/x-www-form-urlencoded\",\n    \"host\": \"postman-echo.com\",\n    \"user-agent\": \"funplugin/0.4.3\",\n    \"x-amzn-trace-id\": \"Root=1-6284c0de-7f630abf452e3ed902142dbb\",\n    \"x-forwarded-port\": \"443\",\n    \"x-forwarded-proto\": \"https\"\n},\n    \"json\": {\n    \"foo1\": \"config_bar1\",\n    \"foo2\": \"bar23\",\n    \"foo3\": \"bar21\"\n},\n    \"url\": \"https://postman-echo.com/post\"\n}",
                                "cookies": {
                                    "sails.sid": "s%3Ad4GCMWZYweUHgfRn3NuhmVbnuXWPu5ZB.4huCHJKsnJdJmcPO06Xbm%2BethQarbyuFJ%2ByGJedYEfo"
                                },
                                "headers": {
                                    "Connection": "keep-alive",
                                    "Content-Length": "471",
                                    "Content-Type": "application/json; charset=utf-8",
                                    "Date": "Wed, 18 May 2022 09:48:14 GMT",
                                    "Etag": "W/\"1d7-vanGqEJBDhXGImIQEVhDyL56LlQ\"",
                                    "Set-Cookie": "sails.sid=s%3Ad4GCMWZYweUHgfRn3NuhmVbnuXWPu5ZB.4huCHJKsnJdJmcPO06Xbm%2BethQarbyuFJ%2ByGJedYEfo; Path=/; HttpOnly",
                                    "Vary": "Accept-Encoding"
                                },
                                "proto": "HTTP/1.1",
                                "status_code": 200
                            }
                        },
                        "validators": [
                            {
                                "check": "status_code",
                                "assert": "eq",
                                "expect": 200,
                                "check_value": 200,
                                "check_result": "pass"
                            },
                            {
                                "check": "body.form.foo1",
                                "assert": "eq",
                                "expect": "config_bar1",
                                "check_value": "config_bar1",
                                "check_result": "pass"
                            },
                            {
                                "check": "body.form.foo2",
                                "assert": "eq",
                                "expect": "bar23",
                                "check_value": "bar23",
                                "check_result": "pass"
                            },
                            {
                                "check": "body.form.foo3",
                                "assert": "eq",
                                "expect": "bar21",
                                "check_value": "bar21",
                                "check_result": "pass"
                            }
                        ]
                    },
                    "content_size": 471
                }
            ],
            "root_dir": ""
        }
    ]
}
```
</details>
