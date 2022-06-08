---
title: Hook 机制（hooks）
weight: 5
description: 使用 Hooks 机制可辅助实现 step 前后置操作
---

## 概念

Hook 机制（hooks）是接口测试中一种常用的机制，用于在测试步骤的开始或结束执行特定的操作，常用于进行辅助日志输出以及资源申请与回收等等。

HttpRunner 支持的 Hook 机制类似于 Python [unittest.TestCase] 类中的 `setUp` 和 `tearDown` 方法，不同的是 HttpRunner 的 `setup_hooks` 和 `teardown_hooks` 字段指定的是一或多个 Hook 函数对应的字符串列表。

## 示例

下面以 YAML 测试用例为例，展示 Hook 机制的写法示例：
 
首先，借助自定义函数机制，在 debugtalk.go 文件中定义 `setup_hook_example` 和 `teardown_hook_example` 两个函数，编译生成 debugtalk.bin 的过程可以参考 [plugin-go] 章节。

```go
package main

import (
	"fmt"
)

func SetupHookExample() string {
	return "setup ..."
}

func SetupShowStepName(args string) string {
	return fmt.Sprintf("step name: %v", args)
}

func SetupShowStepRequest(step_req map[string]interface{}) string {
	return fmt.Sprintf("step request: %v", step_req)
}

func TeardownHookExample() string {
	return "teardown ..."
}

func TeardownShowStepResponse(step_resp interface{}) string {
	return fmt.Sprintf("step response: %v", step_resp)
}

```

之后，在测试用例中指定 `setup_hooks` 和 `teardown_hooks` 字段为自定义 Hook 函数列表，此处采用蛇形命名的风格：

```yaml
config:
  name: hook demo
teststeps:
  - name: get postman-echo
    request:
      method: GET
      url: https://postman-echo.com/get
    setup_hooks:
      - ${setup_hook_example()}
      - ${setup_show_step_name($hrp_step_name)}
      - ${setup_show_step_request($hrp_step_request)}
    teardown_hooks:
      - ${teardown_hook_example()}
      - ${teardown_show_step_response($hrp_step_response)}
```

该测试用例的运行结果如下，重点观察 setup/teardown 的调用部分：

```text
1:43PM INF Set log to color console other than JSON format.
1:43PM ??? Set log level
1:43PM INF [init] SetFailfast failfast=true
1:43PM INF [init] SetSaveTests saveTests=false
1:43PM INF [init] SetRequestsLogOn
1:43PM INF load file path=/Users/bytedance/code/go/httprunner/examples/demo-with-go-plugin/testcases/demo_test.yaml
1:43PM INF load file path=/Users/bytedance/code/go/httprunner/examples/demo-with-go-plugin/.env
1:43PM INF set env key=base_url
1:43PM INF set env key=USERNAME
1:43PM INF set env key=PASSWORD
1:43PM INF load testcases successfully count=1
1:43PM INF load hashicorp go plugin success path=/Users/bytedance/code/go/httprunner/examples/demo-with-go-plugin/debugtalk.bin
1:43PM INF reset session runner
1:43PM INF run testcase start testcase="hook demo"
1:43PM INF reset session runner
1:43PM INF run step start step="get postman-echo" type=request-GET
1:43PM INF function GetNames called on host side
1:43PM INF function GetNames called on host side
1:43PM INF call function via gRPC funcArgs=[] funcName=setuphookexample
1:43PM INF call function success arguments=[] funcName=setup_hook_example output="setup ..."
1:43PM INF function GetNames called on host side
1:43PM INF function GetNames called on host side
1:43PM INF call function via gRPC funcArgs=["get postman-echo"] funcName=setupshowstepname
1:43PM INF call function success arguments=["$hrp_step_name"] funcName=setup_show_step_name output="step name: get postman-echo"
1:43PM INF function GetNames called on host side
1:43PM INF function GetNames called on host side
1:43PM INF call function via gRPC funcArgs=[{"headers":{},"method":"GET","url":"https://postman-echo.com/get"}] funcName=setupshowsteprequest
1:43PM INF call function success arguments=["$hrp_step_request"] funcName=setup_show_step_request output="step request: map[headers:map[] method:GET url:https://postman-echo.com/get]"
-------------------- request --------------------
GET /get HTTP/1.1
Host: postman-echo.com


==================== response ====================
Connected via TLSv1.2
HTTP/1.1 200 OK
Content-Length: 259
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Wed, 08 Jun 2022 05:43:42 GMT
Etag: W/"103-LHLQOgaoe+A4Juu3SbfpSGH1WAE"
Set-Cookie: sails.sid=s%3AAKsr5-UBpaR-yBATWrlWx-HtTgTLdZZ2.%2Fk63x3pwxUrDcnLsRZ4D0OjM2XhM%2FYHmNW8fK18GWuM; Path=/; HttpOnly
Vary: Accept-Encoding

{"args":{},"headers":{"x-forwarded-proto":"https","x-forwarded-port":"443","host":"postman-echo.com","x-amzn-trace-id":"Root=1-62a0370e-7ea78c9710b6320d3b2960d5","user-agent":"Go-http-client/1.1","accept-encoding":"gzip"},"url":"https://postman-echo.com/get"}
--------------------------------------------------
1:43PM INF function GetNames called on host side
1:43PM INF function GetNames called on host side
1:43PM INF call function via gRPC funcArgs=[] funcName=teardownhookexample
1:43PM INF call function success arguments=[] funcName=teardown_hook_example output="teardown ..."
1:43PM INF function GetNames called on host side
1:43PM INF function GetNames called on host side
1:43PM INF call function via gRPC funcArgs=[{"body":{"args":{},"headers":{"accept-encoding":"gzip","host":"postman-echo.com","user-agent":"Go-http-client/1.1","x-amzn-trace-id":"Root=1-62a0370e-7ea78c9710b6320d3b2960d5","x-forwarded-port":"443","x-forwarded-proto":"https"},"url":"https://postman-echo.com/get"},"cookies":{"sails.sid":"s%3AAKsr5-UBpaR-yBATWrlWx-HtTgTLdZZ2.%2Fk63x3pwxUrDcnLsRZ4D0OjM2XhM%2FYHmNW8fK18GWuM"},"headers":{"Connection":"keep-alive","Content-Length":"259","Content-Type":"application/json; charset=utf-8","Date":"Wed, 08 Jun 2022 05:43:42 GMT","Etag":"W/\"103-LHLQOgaoe+A4Juu3SbfpSGH1WAE\"","Set-Cookie":"sails.sid=s%3AAKsr5-UBpaR-yBATWrlWx-HtTgTLdZZ2.%2Fk63x3pwxUrDcnLsRZ4D0OjM2XhM%2FYHmNW8fK18GWuM; Path=/; HttpOnly","Vary":"Accept-Encoding"},"proto":"HTTP/1.1","status_code":200}] funcName=teardownshowstepresponse
1:43PM INF call function success arguments=["$hrp_step_response"] funcName=teardown_show_step_response output="step response: map[body:map[args:map[] headers:map[accept-encoding:gzip host:postman-echo.com user-agent:Go-http-client/1.1 x-amzn-trace-id:Root=1-62a0370e-7ea78c9710b6320d3b2960d5 x-forwarded-port:443 x-forwarded-proto:https] url:https://postman-echo.com/get] cookies:map[sails.sid:s%3AAKsr5-UBpaR-yBATWrlWx-HtTgTLdZZ2.%2Fk63x3pwxUrDcnLsRZ4D0OjM2XhM%2FYHmNW8fK18GWuM] headers:map[Connection:keep-alive Content-Length:259 Content-Type:application/json; charset=utf-8 Date:Wed, 08 Jun 2022 05:43:42 GMT Etag:W/\"103-LHLQOgaoe+A4Juu3SbfpSGH1WAE\" Set-Cookie:sails.sid=s%3AAKsr5-UBpaR-yBATWrlWx-HtTgTLdZZ2.%2Fk63x3pwxUrDcnLsRZ4D0OjM2XhM%2FYHmNW8fK18GWuM; Path=/; HttpOnly Vary:Accept-Encoding] proto:HTTP/1.1 status_code:200]"
1:43PM INF run step end exportVars=null step="get postman-echo" success=true type=request
1:43PM INF run testcase end testcase="hook demo"
1:43PM INF quit hashicorp plugin process
```

## 说明

- 测试步骤运行过程中会保存 `hrp_step_name`、`hrp_step_request` 和 `hrp_step` 三个特殊的变量，类型分别为 string、map[string]interface{} 和 interface{}，上面的示例中展示了在 Hook 函数中传入这三个变量作为参数的方式
- 因为自定义函数本质上会通过 RPC 的方式来进行调用，所以如果使用自定义的函数作为 Hook 函数，则推荐使用返回值的方式来展示自定义函数运行过程的输出，而不是通过常规的日志方式

[unittest.TestCase]: https://docs.python.org/3/library/unittest.html#unittest.TestCase
[plugin-go]: https://httprunner.com/docs/user-guide/enhance-tests/debugtalk/#plugin-go
