---
title: 使用 HttpRunner v4.0 实现 WebSocket 的测试
slug: how-to-test-websocket
date: 2022-05-13
---

HttpRunner 从 v4.0 开始新增支持 WebSocket 协议。

本文将结合案例初步介绍使用 HttpRunner v4.0 测试 WebSocket 的方法，欢迎大家多多实践，后续我们将基于大家的反馈进行迭代优化。

## 功能概览

在 HttpRunner v4.0 中，当前针对 WebSocket 支持了如下能力：

- 提供了 6 种不同的 WebSocket 操作类型，覆盖了 Websocket 接口/性能测试的常见使用场景
- 建立连接阶段支持设置请求参数和请求头
- 支持发送文本/二进制两种类型的消息，支持从本地文件导入的方式来加载二进制消息
- 发送 ping 和 close 控制消息时，同样支持发送文本/二进制消息
- 通知 close 操作通知服务端断开当前连接之前，会自动丢弃现有连接中的冗余消息
- 支持设置代理 URL
- 继承了 HttpRunner 强大的参数关联和结果断言能力

## 案例演示

实践出真知，首先我们来看一个完整的测试用例。

该测试用例使用的被测服务是一个类似于 httpbin 的回显服务，协议类型是 WebSocket 协议：`ws://echo.websocket.events`。无论客户端发送了什么消息，服务端都会原封不动地返回，具体效果可以参考[链接]。

测试用例总共包含了如下的 10 个步骤：

1. 建立一个新的连接并携带请求头 "User-Agent": "HttpRunnerPlus"，并且对响应状态码 `status_code` 和响应头 `headers.Connection` 的内容进行断言
2. 进行一次 ping pong 连接测试，超时时间设为 5 秒
3. 读取建立连接之后，服务端主动推送的赞助消息 "echo.websocket.events sponsored by Lob.com"，并对 `body` 中包含的字符串内容进行断言，超时时间设为 5 秒
4. 写消息，指定消息类型为文本类型，消息内容为 JSON 格式：`{"foo1": "${gen_random_string($n)}", "foo2": "${max($a, $b)}"}`，注意该消息内容中用到了 HttpRunner 的内置函数 `gen_random_string` 和 `max`
5. 读消息，不需要指定消息类型，对 body.foo1 进行参数提取，保存为新的变量 `varFoo1`，随后再对 `body.foo1` 的长度和 `body.foo2` 的值进行断言
6. 读写消息，写消息的类型为文本类型，消息内容为上一步中提取的 `varFoo1`，并对服务端返回的消息进行读取，并且对 body 的长度进行断言
7. 读写消息，写消息的类型为二进制类型，消息内容为从本地文件导入的二进制数据，并对服务端返回的消息进行读取，但不进行任何参数提取和结果断言操作
8. 写一段冗余的消息 "have a nice day!"，用于测试断开连接前冗余消息的自动舍弃
9. 写一段冗余的消息 "balabala ..."，用于测试断开连接前冗余消息的自动舍弃
10. 通知服务端断开当前连接，超时时间设为 30 秒，指定状态码为 1000（说明客户端是正常断开连接），并且对服务端返回的响应状态码进行断言

上述测试用例对应的脚本形式如下：

```yaml
config:
  name: run request with WebSocket protocol
  base_url: ws://echo.websocket.events
  variables:
    a: 12.3
    b: 3.45
    file: "./demo_file_load_ws_message.txt"
    n: 5

teststeps:
- name: open connection
  websocket:
    type: open
    url: "/"
    headers:
      User-Agent: HttpRunnerPlus
  validate:
  - check: status_code
    assert: equals
    expect: 101
    msg: check open status code
  - check: headers.Connection
    assert: equals
    expect: Upgrade
    msg: check headers
- name: ping pong test
  websocket:
    type: ping
    url: "/"
    timeout: 5000
- name: read sponsor info
  websocket:
    type: r
    url: "/"
    timeout: 5000
  validate:
  - check: body
    assert: contains
    expect: Lob.com
    msg: check sponsor message
- name: write json
  websocket:
    type: w
    url: "/"
    text:
      foo1: "${gen_random_string($n)}"
      foo2: "${max($a, $b)}"
- name: read json
  websocket:
    type: r
    url: "/"
  extract:
    varFoo1: body.foo1
  validate:
  - check: body.foo1
    assert: length_equals
    expect: 5
    msg: check json foo1
  - check: body.foo2
    assert: equals
    expect: 12.3
    msg: check json foo2
- name: write and read text
  websocket:
    type: wr
    url: "/"
    text: "$varFoo1"
  validate:
  - check: body
    assert: length_equals
    expect: 5
    msg: check length equal
- name: write and read binary file
  websocket:
    type: wr
    url: "/"
    binary: "${load_ws_message($file)}"
- name: write something redundant
  websocket:
    type: w
    url: "/"
    text: have a nice day!
- name: write something redundant
  websocket:
    type: w
    url: "/"
    text: balabala ...
- name: close connection
  websocket:
    type: close
    url: "/"
    close_status: 1000
    timeout: 30000
  validate:
  - check: status_code
    assert: equals
    expect: 1000
    msg: check close status code
```

需要说明的是，该测试用例中的示例文件需要由用户自己来指定，文件路径填写有效的绝对路径或相对路径即可，这里的示例文件  `demo_file_load_ws_message.txt`  只是以 txt 文件为例，实际上该文件可以为任意类型，最终导入之后都为二进制类型。

除了 YAML/JSON 格式外，我们也可以采用 gotest 的方式编写用例，形式如下：

```go
package tests

import (
   "testing"

   "github.com/httprunner/httprunner/v4/hrp"
)

func TestWebSocketProtocol(t *testing.T) {
   testcase := &hrp.TestCase{
      Config: hrp.NewConfig("run request with WebSocket protocol").
         SetBaseURL("ws://echo.websocket.events").
         WithVariables(map[string]interface{}{
            "n":    5,
            "a":    12.3,
            "b":    3.45,
            "file": "./demo_file_load_ws_message.txt",
         }),
      TestSteps: []hrp.IStep{
         hrp.NewStep("open connection").
            WebSocket().
            OpenConnection("/").
            WithHeaders(map[string]string{"User-Agent": "HttpRunnerPlus"}).
            Validate().
            AssertEqual("status_code", 101, "check open status code").
            AssertEqual("headers.Connection", "Upgrade", "check headers"),
         hrp.NewStep("ping pong test").
            WebSocket().
            PingPong("/").
            WithTimeout(5000),
         hrp.NewStep("read sponsor info").
            WebSocket().
            Read("/").
            WithTimeout(5000).
            Validate().
            AssertContains("body", "Lob.com", "check sponsor message"),
         hrp.NewStep("write json").
            WebSocket().
            Write("/").
            WithTextMessage(map[string]interface{}{"foo1": "${gen_random_string($n)}", "foo2": "${max($a, $b)}"}),
         hrp.NewStep("read json").
            WebSocket().
            Read("/").
            Extract().
            WithJmesPath("body.foo1", "varFoo1").
            Validate().
            AssertLengthEqual("body.foo1", 5, "check json foo1").
            AssertEqual("body.foo2", 12.3, "check json foo2"),
         hrp.NewStep("write and read text").
            WebSocket().
            WriteAndRead("/").
            WithTextMessage("$varFoo1").
            Validate().
            AssertLengthEqual("body", 5, "check length equal"),
         hrp.NewStep("write and read binary file").
            WebSocket().
            WriteAndRead("/").
            WithBinaryMessage("${load_ws_message($file)}"),
         hrp.NewStep("write something redundant").
            WebSocket().
            Write("/").
            WithTextMessage("have a nice day!"),
         hrp.NewStep("write something redundant").
            WebSocket().
            Write("/").
            WithTextMessage("balabala ..."),
         hrp.NewStep("close connection").
            WebSocket().
            CloseConnection("/").
            WithTimeout(30000).
            WithCloseStatus(1000).
            Validate().
            AssertEqual("status_code", 1000, "check close status code"),
      },
   }

   // run testcase
   err = hrp.NewRunner(t).Run(testcase)
   if err != nil {
      t.Fatalf("run testcase error: %v", err)
   }
}
```

可以看出，WebSocket 协议的脚本形式与之前 HTTP(S) 协议的格式基本一致。如果大家使用 gotest 编写脚本，也可以借助「链式调用」获得方法提示。

## 操作类型介绍

WebSocket 不同于 HTTP 协议，不再使用 GET/POST/PUT/etc. 请求方法，而是包含 `open/close/ping/w/r/wr` 6 种操作类型。

下面对这 6 种操作类型进行介绍：

- open：创建一个新的 WebSocket 连接，创建连接是进行通信的第一步，如果在进行后续操作之前不先创建连接，则执行时会因为当前无可用连接而报错。WebSocket 的全双工连接最初是客户端发起 HTTP 请求，并进行协议升级而得来的，所以在建立连接的阶段是可以像常规的 HTTP 请求一样设置请求参数和请求头的，HttpRunner 支持在创建连接阶段设置请求参数和请求头，从而可以满足更加灵活的测试场景。需要注意的是，进行协议升级时的请求方法只限于 GET 方法，如果为 POST 方法，则请求会被 HTTP 服务而不是 WebSocket 服务处理。open 操作除了支持设置请求参数和请求头之外，还支持设置超时时间，这里的超时时间可以理解为是握手阶段的超时时间
- ping：客户端发送一个 ping 控制消息，并且期待服务端返回的 pong 控制消息。WebSocket 建立的连接会占用客户端和服务端双方的网络资源，因此服务端一般会在无新操作之后的一段时间自动关闭当前连接，例如上面例子中用到的 echo.websocket.events 服务就会在双方无操作 60 秒之后自动关闭连接。如果客户端希望保持当前连接处于不断开的状态，又不希望发送具体的文本/二进制消息，则可以定时发送一个 ping 控制消息来维持当前连接，服务端会在收到 ping 控制消息之后向客户端回传一个 pong 控制消息，客户端接受到 pong 控制消息则可以确认服务端会继续保持当前连接的状态。上述的过程又称为 WebSocket 心跳机制，HttpRunner 支持在 ping 操作的同时发送文本/二进制消息，同时接收 pong 的过程是异步进行的，不会阻塞后续步骤的进行的，然而也正是由于接收 pong 的过程是异步进行的，暂不支持对服务端的 pong 响应进行参数提取和结果断言
- w：客户端尝试向服务端进行一次写消息，适用于性能测试中单独测试上行通信的场景，该操作暂不支持设置超时时间。写消息的内容可以为文本/二进制消息，如果为二进制消息则可以通过 HttpRunner 的内置函数 `load_ws_message` 来导入消息内容。由于写消息的过程是客户端向服务端的单向通信过程，不存在接收响应的过程，因此也不支持响应的参数提取和结果断言
- r：客户端尝试对服务端发来的消息进行一次读取，适用于性能测试中单独测试下行通信的场景，需要注意读取消息的内容仅限于文本/二进制消息，控制消息是在 ping / close 操作中单独来接收和处理的
- wr：进行一次消息读写，可以理解为是 w 和 r 两个操作的顺序组合，其过程类似于 HTTP 协议中发送请求和接收响应的过程，是 WebSocket 接口测试中较为常见的一类操作
- close：通知服务端断开当前的 WebSocket 连接，并且在断开前，当前连接中的冗余消息会被舍弃。与 ping 操作类似，在发送 close 控制消息的同时也支持发送文本/二进制消息，此外还支持指定断开连接的状态码，该状态码可以用于说明断开连接的原因，不同状态码的含义可以参考 [RFC 6455](https://www.rfc-editor.org/rfc/rfc6455.html#section-11.7) 中的详细介绍

需要注意的是，6 种操作类型会存在一些限制差异，可查看如下对比表格：

|     操作类型     | open | ping |  w  |  r  | wr  | close |
|:------------:|:----:|:----:|:---:|:---:|:---:|:-----:|
|   支持设置超时时间   |  ✅   |  ✅   |  ❌  |  ✅  |  ✅  |   ✅   |
| 支持设置请求参数和请求头 |  ✅   |  ❌   |  ❌  |  ❌  |  ❌  |   ❌   |
|   是否为阻塞操作    |  ✅   |  ❌   |  ✅  |  ✅  |  ✅  |   ✅   |
|    支持发送消息    |  ❌   |  ✅   |  ✅  |  ❌  |  ✅  |   ✅   |
|   支持设置状态码    |  ❌   |  ❌   |  ❌  |  ❌  |  ❌  |   ✅   |
| 支持参数提取和结果断言  |  ✅   |  ❌   |  ❌  |  ✅  |  ✅  |   ✅   |

## 注意事项

另外，在测试 WebSocket 协议的时候还需注意如下事项：

- 在参数未填写或者非法的情况下，所有操作的超时时间默认为 30 秒，close 操作的状态码默认为 1000（正常断开）
- 不同类型的 WebSocket 操作支持的参数提取和结果断言的对象类型也是不同的：
    - open 操作支持的参数提取和结果断言的对象类型与 HTTP 协议支持的相同，包括 proto、status_code、headers、cookies 和 body
    - r 和 wr 操作支持的参数提取和结果断言的对象类型只有 body
    - close 操作支持的参数提取和结果断言的对象类型包括 status_code 和 body，并且 body 的类型一定为字符串
- 发送二进制消息时，建议使用内置函数 `load_ws_message` 从本地文件导入的方式来加载二进制消息，因为通过 JSON/YAML 脚本的方式无法直接指定二进制消息的内容（采用 go test 的方式来编写测试用例时，也可以通过 `bytes.Buffer` 的方式来指定二进制消息内容）
- 如果测试用例的最后一步没有指定 close 操作，当前会话结束时也会自动断开当前连接，但是断开前不会丢弃当前连接中的冗余消息，也不会通知服务端断开连接，而是从客户端直接断开连接。

## What's next

- 目前 hrp har2case 命令还不支持将包含 WebSocket 请求的 har 包转换为 JSON/YAML 测试用例，HttpRunner v4.0 之后的版本中将会对该特性进行支持
- 目前暂不支持自动重连机制，之后版本将支持在服务端主动断开连接后进行自动重连，并支持指定自动重连的间隔时间和最大重试次数
- 目前二进制消息会转为字符串或者程序内部变量的形式来进行参数提取和结果断言的，后续计划支持二进制消息的单独参数提取和结果断言机制
- 单独维护一个 WebSocket 消息缓冲区，将 ping 操作的执行过程由异步调整为同步，从而可以对 pong 响应的结果进行参数提取和断言

> 本文作者：卜卜星，HttpRunner 核心开发者，贡献了本文介绍的 WebSocket 协议测试能力<br/>
> HttpRunner 项目官网: https://httprunner.com/<br/>
> 如果 HttpRunner 对你有过帮助，麻烦帮忙给个 ⭐️star⭐️ 鼓励下吧<br/>
> https://github.com/httprunner/httprunner

[链接]: https://echo.websocket.events/.ws
[RFC 6455]: https://www.rfc-editor.org/rfc/rfc6455.html#section-11.7
