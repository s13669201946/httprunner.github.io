---
title: 手工编写
weight: 3
description: 手工编写 YAML/JSON/pytest/gotest 测试用例
---

## 通用概念介绍

在介绍如何手工编写测试用例之前，首先介绍 HttpRunner 中的一些通用概念，理解这些概念对于手工编写测试用例至关重要，因为无论是哪种形态的测试用例，都是按照一定的规则来设计的。这些通用概念具体包含以下的内容：

### 测试步骤（teststep）

测试步骤是测试用例的最小执行单元，测试用例是测试步骤的有序集合，对于接口测试来说，每一个测试步骤应该就对应一个具体的用户操作。HttpRunner 支持的测试步骤如下：

| 测试步骤类型      | 含义                     |
|-------------|------------------------|
| request     | 用于发起 HTTP 请求的步骤类型      |
| api         | 用于引用 API 的步骤类型         |
| testcase    | 用于引用其他测试用例的步骤类型        |
| transaction | 用于定义一个事务               |
| rendezvous  | 集合点                    |
| think_time  | 思考时间                   |
| websocket   | 用于发起 WebSocket 请求的步骤类型 |

除了基本的测试步骤之外，部分测试步骤还可以进行增强，从而拓展更多功能，这一部分内容将在[增强测试用例]章节中进行详细介绍。

| 增强操作类型         | 含义   | 适用的测试步骤               |
|----------------|------|-----------------------|
| variables      | 局部变量 | 通用                    |
| setup_hooks    | 前置函数 | request/api/websocket |
| teardown_hooks | 后置函数 | request/api/websocket |
| extract        | 参数提取 | request/api/websocket |
| validate       | 结果校验 | request/api/websocket |
| export         | 导出变量 | testcase              |

### 测试用例（testcase）

从 2.0 版本开始，HttpRunner 开始对测试用例的概念进行进一步的明确，参考 wiki 上对于 [test case] 的定义：

> A test case is a specification of the inputs, execution conditions, testing procedure, and expected results that define a single test to be executed to achieve a particular software testing objective, such as to exercise a particular program path or to verify compliance with a specific requirement.

概括下来，一条测试用例（testcase）应该是为了测试某个特定的功能逻辑而精心设计的，并且至少包含如下几点：

- 明确的测试目的（achieve a particular software testing objective）
- 明确的输入数据（inputs）
- 明确的运行环境（execution conditions）
- 明确的测试步骤描述（testing procedure）
- 明确的预期结果（expected results）

按照上述的测试用例定义，HttpRunner 的测试用例应该保证是完整并且可以独立运行的。

从测试用例的组成结构来看，一个测试用例可以分为「测试脚本」和「测试数据」两部分：
- 测试脚本：重点是描述测试的业务功能逻辑，包括预置条件、测试步骤、预期结果等，并且可以结合辅助函数（debugtalk.go/debugtalk.py）实现复杂的运算逻辑
- 测试数据：重点是对应测试的业务数据逻辑，例如数据驱动文件中的定义的 UUID、用户名等等，以及环境配置文件中定义的 base_url 环境变量等等

测试脚本与测试数据分离后，就可以比较方便地实现数据驱动测试，通过对测试脚本传入一组测试数据，实现同一业务功能在输入不同业务数据情况下的测试验证。那么测试数据是如何传入测试脚本中的呢？这就需要用到测试数据与测试脚本之间的桥梁——用例配置（config），测试脚本除了包含多个有序的测试步骤之外，还包含针对当前测试用例的用例配置项，HttpRunner 支持的用例配置项如下：

| 用例配置项名称            | 含义                                             |
|--------------------|------------------------------------------------|
| verify             | 客户端是否进行 SSL 校验（todo）                           |
| base_url           | 当前请求的基础 URL（deprecated in v4.1, moved to .env） |
| headers            | 定义测试用例级别的请求体                                   |
| environs           | 配置环境变量（如果未指定则会从 .env 文件导入）                     |
| variables          | 定义测试用例级别的变量                                    |
| parameters         | 配置参数驱动的数据源                                     |
| parameters_setting | 配置参数驱动的具体策略                                    |
| think_time         | 配置思考时间的具体策略、超时时间限制等                            |
| websocket          | 配置 WebSocket 断开重连的最大次数和间隔等（todo）               |
| export             | 导出当前测试用例中的变量                                   |
| weight             | 性能测试过程中，分配给当前测试用例的虚拟用户权重                       |
| path               | 当前测试用例所在路径（通常不需要手工填写）                          |

### 测试用例集（testsuite）

对于测试用例集的概念，同样参考 wiki 上对于 [test suite] 的定义：

> a test suite, less commonly known as a validation suite, is a collection of test cases that are intended to be used to test a software program to show that it has some specified set of behaviors. A test suite often contains detailed instructions or goals for each collection of test cases and information on the system configuration to be used during testing. A group of test cases may also contain prerequisite states or steps, and descriptions of the following tests.

从上面定义中可以看出，测试用例集是测试用例的无序集合，通俗来将，测试用例集就是「一堆」测试用例。对应地，HttpRunner 除了支持指定单个文件来运行某一测试用例，也支持指定多个文件或指定文件夹来运行一整个测试用例集。

### 项目根目录：

项目根目录是整个测试用例集的目录，测试用例中的相对路径（例如在测试步骤中引用 API 或测试用例的路径，或者在使用参数驱动功能时引用 CSV 文件的路径）都是基于项目根目录来进行引用的。在 HttpRunner v4.0 中，项目根目录以 proj.json 为锚，并且按照使用习惯，环境配置文件 .env，以及辅助函数的定义文件 debugtalk.py/debugtalk.go 通常也是放在项目根目录下的。

## 手工编写 YAML/JSON 测试用例

YAML 和 JSON 是两种常用的定义结构化数据的格式，关于这两种格式的优劣是非常有争议的问题，下面首先从编写风格、功能多样性和解析效率几个方面进行一下对比：

- 编写风格：YAML 必须使用空格来表示缩进，并且对空格具体数量没有要求，只需要相同层级左侧对齐即可，这种缩进风格简洁且易于阅读，针对同一测试用例，YAML 文件的篇幅通常要比带缩进的 JSON 文件简短很多，用户可以较快地把握整体结构，但是对于手动编写 YAML 测试用例的场景而言，则需要重点注意相同层级左侧对齐，否则很容易导致测试用例的加载错误。JSON 的编写风格则是频繁地使用 {} [] 和 ""，对缩进并没有严格要求（习惯上还是采用良好的缩进风格来便于阅读），正是由于额外的符号过多，容易降低阅读和手工编写上的体验
- 功能多样性：在功能方面，YAML 是 JSON 的超集，YAML 有许多 JSON 缺乏的附加功能，包括注释、可扩展数据类型、关系锚、不带引号的字符串等等，尤其是添加注释的功能在编写测试用例供他人阅读时尤为实用，而 JSON 只支持数值、字符串、布尔值、数组、对象和空值六种数据类型。
- 解析效率：与 YAML 相比，JSON 对机器来说更容易解析，序列化/反序列化的速度更快

从上面的对比中可以看出 YAML 与 JSON 各有优劣，用户在编写测试用例的时可以根据自己的喜好和使用场景来进行选择，不过二者具有较强的相似性，下面以 YAML 为例来对手工编写测试用例进行介绍，JSON 测试用例的编写方式同理，只需要适配 JSON 的语法规则即可。

### 编写用例配置

下面展示一个 YAML 用例配置部分的示例，读者只需要了解 YAML 的基本语法规则以及各个字段的类型即可在项目中自定义用例配置部分。

```yaml
config:
  name: "demo with complex mechanisms"
  verify: False
  base_url: "https://postman-echo.com"
  headers:
    X-Request-Timestamp: "165460624942"
  parameters:
    user_agent: [ "iOS/10.1", "iOS/10.2" ]
    username-password: ${parameterize($file)}
  parameters_setting:
    strategies:
      user_agent:
        name: "user-identity"
        pick_order: "sequential"
      username-password:
        name: "user-info"
        pick_order: "random"
    limit: 6
  think_time:
    strategy: random_percentage
    setting:
      max_percentage: 1.5
      min_percentage: 1
    limit: 4
  variables:
    app_version: v1
    user_agent: iOS/10.3
    file: examples/hrp/account.csv
  websocket:
    reconnection_times: 5
    reconnection_interval: 2000
  export: ["app_version"]
  weight: 10
```

### 编写测试步骤

#### request
```yaml
teststeps:
  -
    name: get with params
    variables:
      foo1: ${ENV(USERNAME)}
      foo2: bar21
      sum_v: "${sum_two_int(1, 2)}"
    request:
      method: GET
      url: $base_url/get
      params:
        foo1: $foo1
        foo2: $foo2
        sum_v: $sum_v
    extract:
      foo3: "body.args.foo2"
    validate:
      - eq: ["status_code", 200]
      - eq: ["body.args.foo1", "debugtalk"]
      - eq: ["body.args.sum_v", "3"]
      - eq: ["body.args.foo2", "bar21"]
  -
    name: post raw text
    variables:
      foo1: "bar12"
      foo3: "bar32"
    request:
      method: POST
      url: $base_url/post
      headers:
        Content-Type: "text/plain"
      body: "This is expected to be sent back as part of response body: $foo1-$foo2-$foo3."
    validate:
      - eq: ["status_code", 200]
      - eq: ["body.data", "This is expected to be sent back as part of response body: bar12-$expect_foo2-bar32."]
  -
    name: post form data
    variables:
      foo2: bar23
    request:
      method: POST
      url: $base_url/post
      headers:
        Content-Type: "application/x-www-form-urlencoded"
      body: "foo1=$foo1&foo2=$foo2&foo3=$foo3"
    validate:
      - eq: ["status_code", 200]
      - eq: ["body.form.foo1", "$expect_foo1"]
      - eq: ["body.form.foo2", "bar23"]
      - eq: ["body.form.foo3", "bar21"]
```

HTTP/2 请求请参考 [HTTP/2] 章节

#### api

```yaml
teststeps:
    - name: test api /get
      api: api/get.json
      variables:
        app_version: 2.8.7
        device_sn: $device_sn
        os_platform: ios
        user_agent: iOS/10.4
      extract:
        session_token: body.headers."postman-token"
      validate:
        - check: status_code
          assert: equal
          expect: 200
          msg: check status_code
        - check: body.headers."postman-token"
          assert: equal
          expect: ea19464c-ddd4-4724-abe9-5e2b254c2723
          msg: check body.headers.postman-token
```

#### testcase

```yaml
teststeps:
-
    name: request with functions
    variables:
        foo1: testcase_ref_bar1
        expect_foo1: testcase_ref_bar1
    testcase: testcases/requests.yml
    export:
        - foo3
```

#### transaction

请参考[事务（transaction）]章节

#### rendezvous

请参考[集合点（rendezvous）]章节

#### think_time

请参考[思考时间（think_time）]章节

#### websocket

请参考 [WebSocket] 章节


### 注意事项

与下面即将介绍的代码形态编写测试用例的方式不同，手工编写 YAML/JSON 测试用例具有较大的灵活性并且容易出错，因此在手工编写测试用例的过程中需要格外注意。最后，步骤类型的判断是会有一定优先级的，如果用户在一个测试步骤中指定了多个步骤类型，则会按照 api > testcase > think_time > request > transaction > rendezvous > websocket 的顺序进行类型识别，并最终判断为其中的一类，虽然这种情况极少会出现，但也在此说明一下，以防止发生一些意料之外的错误

## 手工编写 Pytest 测试用例
## 手工编写 gotest 测试用例

[test case]: https://en.wikipedia.org/wiki/Test_case
[test suite]: https://en.wikipedia.org/wiki/Test_suite
[增强测试用例]: https://httprunner.com/docs/user-guide/enhance-tests/
[HTTP/2]: https://httprunner.com/docs/user-guide/protocols/h2/
[事务（transaction）]: https://httprunner.com/docs/user-guide/load-test/concepts/#%E4%BA%8B%E5%8A%A1transaction
[集合点（rendezvous）]: https://httprunner.com/docs/user-guide/load-test/concepts/#%E9%9B%86%E5%90%88%E7%82%B9rendezvous
[思考时间（think_time）]: https://httprunner.com/docs/user-guide/load-test/concepts/#%E6%80%9D%E8%80%83%E6%97%B6%E9%97%B4think_time
[WebSocket]: https://httprunner.com/docs/user-guide/protocols/websocket/