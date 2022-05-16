---
title: 如何使用 HttpRunner v4.0 开展性能测试
slug: how-to-load-test
date: 2022-05-16
---

## 前言

在 [HttpRunner v4.0 全新发布] 中我们有介绍过，HttpRunner v4.0 期望成为一款专业级的一体化 API 测试工具，特别是针对性能测试能力进行了重大升级。相比于之前的版本，HttpRunner v4.0 在性能测试部分最大的优化包括如下 4 个方面：

- 使用 Golang 重新实现了脚本执行引擎（基于 [Boomer]），相比于 Python Locust 极大地提升了发压能力
- 对标 LoadRunner 新增实现了丰富的性能测试机制，包括事务、集合点、思考时间等
- 对压测结果进行了严格的 benchmark 测试和数据校准，确保测试结果真实可靠
- 集成了 Prometheus 性能采集能力，配合 Grafana 可实现丰富的性能指标看板

涉及的内容比较多，因此我们针对 HttpRunner v4.0 的性能测试能力规划一系列专题文章，包括性能测试工具使用、监控配置、benchmark 数据比对、竞品工具对比、原理解析等等。

本文作为性能测试专题的第一篇文章，将结合一个简单的案例整体介绍如何使用 HttpRunner v4.0 开展性能测试，帮助大家快速上手使用工具。

## 案例介绍

为了方便大家理解，本文挑选了一个非常简单的案例场景，但同时会尽量多地覆盖常用的性能测试特性。

案例设计如下：

- 压测目标为 3 个接口：API1/API2/API3
- 业务层面期望整体关注 API1 + API2 的性能情况，即「事务」特性
- 在真实场景中，API1 和 API2 请求之间需要有一个间隔时间，即「思考时间」特性
- 期望重点关注 API3 的并发性能，即所有用户同一时间请求的情况，即「集合点」特性

接下来，我们将对这些特性进行具体介绍，并对使用方法进行演示说明。

## 编写测试用例

在性能测试之前，我们需要先准备好性能测试用例。在 HttpRunner 中，得益于「一体化」的特性优势，我们可以在无需对已有接口测试用例做任何修改的情况下，直接运行性能测试。

针对本文中的案例，接口测试用例如下所示：

```yaml
config:
  name: load test demo
  variables:
    app_version: v1
    user_agent: iOS/10.3
  base_url: 'http://httpbin.org'
  verify: false
teststeps:
  - name: get with params
    request:
      method: GET
      url: /get
      headers:
        User-Agent: '$user_agent/$app_version'
    validate:
      - check: status_code
        assert: equals
        expect: 200
        msg: check status code
  - name: post with params
    request:
      method: POST
      url: /post
      headers:
        User-Agent: '$user_agent/$app_version'
    validate:
      - check: status_code
        assert: equals
        expect: 200
        msg: check status code
  - name: post with params 2
    request:
      method: POST
      url: /post
      headers:
        User-Agent: '$user_agent/$app_version'
    validate:
      - check: status_code
        assert: equals
        expect: 200
        msg: check status code
```

我们只需将测试命令从 `hrp run` 改为 `hrp boom`，即可启动性能测试，具体的参数配置后面会详细介绍。

需要说明的是：

- HttpRunner v4.0 做性能测试时，测试用例格式只能选择 YAML/JSON/GoTest；PyTest 格式的底层引擎基于 pytest 执行引擎，不再支持性能测试；
- 如果我们在性能测试中期望实现「事务」、「集合点」、「思考时间」等机制，需要在接口测试用例基础上添加特定的步骤。

### 添加「事务」机制

事务可以将多个接口的测试结果进行聚合统计，性能指标更加贴合真实业务场景。

事务的数据结构如下：

```go
type Transaction struct {
   Name string          `json:"name" yaml:"name"` // 事务名称，可定义为任意字符串
   Type transactionType `json:"type" yaml:"type"` // 事务类型，仅包括 2 种类型，start（事务开始）和 end（结束事务）
}
```

事务使用示例：

```yaml
config:
  ...
teststeps:
  ...
  - name: transaction 1 start
    transaction:
      name: tran1
      type: start
  ...
  - name: transaction 1 end
    transaction:
      name: tran1
      type: end
  ...
```

针对「事务」机制，有 2 个需要特别注意的点：

- 使用 HttpRunner v4.0 执行性能测试时，会自动添加名称为 Action 的事务，该事务包含整个测试用例的所有测试步骤（参考了 LoadRunner 的做法）
- 在测试用例中，transaction 应该成对出现，即必须同时定义 start 和 end；如果存在配对缺失的情况，会按照如下逻辑进行处理：
  - 仅设置开始事务，则会在测试用例最后一个测试步后添加结束事务
  - 仅设置结束事务，则会在测试用例第一个测试步前添加开始事务

### 添加「集合点」机制

集合点可以确保指定的虚拟用户在同一时刻发起请求，实现类似”秒杀“的真实业务场景。

集合点的数据结构如下：

```go
type Rendezvous struct {
   Name           string  `json:"name" yaml:"name"`
   Percent        float32 `json:"percent,omitempty" yaml:"percent,omitempty"`
   Number         int64   `json:"number,omitempty" yaml:"number,omitempty"`
   Timeout        int64   `json:"timeout,omitempty" yaml:"timeout,omitempty"`
}
```

集合点数据结构参数说明如下：

- name: 集合点名称，可定义为任意字符串
- percent: 触发集合点释放的虚拟用户百分比，范围为 (0, 1]，默认为 1，即全部用户
- number: 触发集合点释放的虚拟用户数量，范围为 (0, N]，N 为当前性能测试的所有并发用户数
- timeout: 虚拟用户到达集合点之间的间隔时间的最大值，单位为毫秒；当最近两个虚拟用户达到集合点的时间间隔大于最大值，则立即释放所有虚拟用户，如果没有指定，默认值为 5000 ms

集合点使用示例：

```yaml
config:
  ...
teststeps:
  ...
  - name: rendezvous 1
    rendezvous:
      name: rendezvous1
      number: 50
      timeout: 3000
  ...
  - name: rendezvous 2
    rendezvous:
      name: rendezvous2
      percent: 0.8
      timeout: 3000
  ...
```

针对「集合点」机制，有 2 个需要特别注意的点：

- number 和 percent 参数需要二选一并且保证合法；如果没有指定参数、同时指定了两个参数、或者参数不合法，则默认为需要全部虚拟用户到达集合点。
- 集合点仅在虚拟用户全部加载完之后的稳定阶段生效。

### 添加「思考时间」机制

思考时间可以模拟用户在不同操作间的停顿时间，最大程度还原用户真实的操作行为。

思考时间的数据结构如下：

```go
type ThinkTimeConfig struct {
   Strategy thinkTimeStrategy `json:"strategy,omitempty" yaml:"strategy,omitempty"`
   Setting  interface{}       `json:"setting,omitempty" yaml:"setting,omitempty"`
   Limit    float64           `json:"limit,omitempty" yaml:"limit,omitempty"`
}

type ThinkTime struct {
   Time float64 `json:"time" yaml:"time"`
}
```

其中，`ThinkTimeConfig` 作为「思考时间」的整体策略，需要在测试用例的 `Config` 中配置，参数说明如下：

- Strategy (支持如下四种策略)
  - default: 默认策略，会保持测试用例中设置的思考时间
  - random_percentage: 测试用例中设置的思考时间在指定放缩区间中随机选值
  - multiply: 对测试用例中设置的思考时间进行放缩
  - ignore: 忽略测试用例中设置的思考时间
- Setting (仅需random_percentage、multiply策略下配置，其他模式自动忽略此设置)
  - random_percentage：此策略下，需设置思考时间的最小最大放缩比：min_percentage、max_percentage（设置类型为map），默认: [0.5, 1.5]
  - multiply：此策略下，直接设置放缩比例（设置类型为float），默认: 1
- Limit
  - 对所有策略有效，如果设置且大于0，则限制最终的思考时间<=设定值，默认: -1(无限制)

`ThinkTime` 作为具体步骤间的「思考时间」配置，以单独的 step 存在，参数说明如下：

- time: 不同测试步间的思考时间，单位: 秒

思考时间使用示例如下：

```yaml
config:
  ...
  think_time:
    strategy: random_percentage
    setting:
      min_percentage: 1
      max_percentage: 1.5
    limit: 4
  ...
teststeps:
  ...
  - name: think time 1
    think_time:
      time: 2
  ...
  - name: think time 2
    think_time:
      time: 3
  ...
```

在该示例中，思考时间策略为随机比例（`random_percentage`），介于 100% ~ 150% 之间；limit 设置为 4。经过换算，测试步骤中的 `think time 1` 的最终值在 [2, 3] 之间，`think time 2` 的最终值在 [3, 4] 之间。

### 用例格式示例

经过对已有的接口测试用例进行增强，我们获得了包含「事务」、「集合点」、「思考时间」机制的性能测试用例。

YAML 用例格式的示例如下所示：

```yaml
config:
  name: load test demo
  variables:
    app_version: v1
    user_agent: iOS/10.3
  base_url: 'http://httpbin.org'
  think_time:
    strategy: random_percentage
    setting:
      min_percentage: 1
      max_percentage: 1.5
    limit: 2
  verify: false
teststeps:
  - name: transaction 1 start
    transaction:
      name: tran1
      type: start
  - name: get with params
    request:
      method: GET
      url: /get
      headers:
        User-Agent: '$user_agent,$app_version'
    validate:
      - check: status_code
        assert: equals
        expect: 200
        msg: check status code
  - name: transaction 1 end
    transaction:
      name: tran1
      type: end
  - name: think time 1
    think_time:
      time: 1.5
  - name: post with params
    request:
      method: POST
      url: /post
      headers:
        User-Agent: '$user_agent,$app_version'
    validate:
      - check: status_code
        assert: equals
        expect: 200
        msg: check status code
  - name: rendezvous 1
    rendezvous:
      name: rendezvous1
      percent: 0.8
      timeout: 3000
  - name: post with params 2
    request:
      method: POST
      url: /post
      headers:
        User-Agent: '$user_agent,$app_version'
    validate:
      - check: status_code
        assert: equals
        expect: 200
        msg: check status code
```

对应的 GoTest 用例格式示例如下：

```go
package tests

import (
   "testing"
   "time"

   "github.com/httprunner/httprunner/v4/hrp"
)

func TestLoadTestDemo(t *testing.T) {
   testcase1 := &hrp.TestCase{
      Config: hrp.NewConfig("Load Test Demo").
         SetBaseURL("http://httpbin.org").
         SetThinkTime(
            "random_percentage",
            map[string]float64{"min_percentage": 1, "max_percentage": 1.5},
            4),
      TestSteps: []hrp.IStep{
         hrp.NewStep("transation 1 start").
            StartTransaction("trans1"),
         hrp.NewStep("headers").
            GET("/headers").
            Validate().
            AssertEqual("status_code", 200, "check status code").
            AssertEqual("headers.\"Content-Type\"", "application/json", "check http response Content-Type"),
         hrp.NewStep("transation 1 end").
            EndTransaction("trans1"),
         hrp.NewStep("think time1").
            SetThinkTime(3),
         hrp.NewStep("user-agent").
            GET("/user-agent").
            Validate().
            AssertEqual("status_code", 200, "check status code").
            AssertEqual("headers.\"Content-Type\"", "application/json", "check http response Content-Type"),
         hrp.NewStep("rendezvous 1").
            SetRendezvous("rend1").
            WithUserPercent(0.8).
            WithTimeout(3000),
         hrp.NewStep("TestCaseRef").
            CallRefCase(&hrp.TestCase{Config: hrp.NewConfig("TestCase2")}),
      },
   }

   b := hrp.NewStandaloneBoomer(1000, 100) //spawn_count: 1000, spawn_rate: 100
   go b.Run(testcase1)
   time.Sleep(1000 * time.Second) //expected running time
   b.Quit()
}
```

## 性能测试参数配置

准备好性能测试用例后，我们就可以开始执行性能测试了。

当前 HttpRunner v4.0 支持多种性能测试策略，我们可以在命令行参数中进行指定。

关于性能测试的参数信息，可以通过 `hrp boom -h` 进行查看。

```text
$ hrp boom --help
run yaml/json testcase files for load test

Usage:
  hrp boom [flags]

Examples:
  $ hrp boom demo.json        # run specified json testcase file
  $ hrp boom demo.yaml        # run specified yaml testcase file
  $ hrp boom examples/        # run testcases in specified folder

Flags:
      --cpu-profile string              Enable CPU profiling.
      --cpu-profile-duration duration   CPU profile duration. (default 30s)
      --disable-compression             Disable compression
      --disable-console-output          Disable console output.
      --disable-keepalive               Disable keepalive
  -h, --help                            help for boom
      --loop-count int                  The specify running cycles for load testing (default -1)
      --max-rps int                     Max RPS that boomer can generate, disabled by default.
      --mem-profile string              Enable memory profiling.
      --mem-profile-duration duration   Memory profile duration. (default 30s)
      --prometheus-gateway string       Prometheus Pushgateway url.
      --request-increase-rate string    Request increase rate, disabled by default. (default "-1")
      --spawn-count int                 The number of users to spawn for load testing (default 1)
      --spawn-rate float                The rate for spawning users (default 1)

Global Flags:
      --log-json           set log to json format
  -l, --log-level string   set log level (default "INFO")
```

参数看上去比较多，接下来对其按照用途进行分组介绍。

### 设置并发用户数

性能测试需要指定并发用户数，同时在很多时候我们需要按照阶梯加压的方式初始化并发用户。这里就需要使用到 `--spawn-count` 和 `--spawn-rate` 两个参数了。

- spawn-count：表示我们期望达到的最大并发用户数
- spawn-rate：表示达到期望最大并发前每秒增加的并发用户数

具体示例如下所示：

```bash
$ hrp boom testcase.yml --spawn-count 1000 --spawn-rate 100
```

在这种模式下，性能测试不会自动结束；当我们期望结束压测时，可以通过 kill 命令杀死进程，或者 `ctrl+c` 来结束性能测试，结束时终端会打印整体测试数据。

### 设置 RateLimiter

有时我们需要在性能测试时对发压流量进行限流，例如期望设置最大 RPS，或者在初始化并发用户时限定增加数率。这就需要使用到 `--max-rps` 和 `--request-increase-rate`。

- max-rps：限制最大 rps，默认不限制
- request-increase-rate：限制每秒的请求增加速率，默认不开启，支持两种格式："1"、"1/1s"

具体示例如下所示：

```bash
$ hrp boom testcase.yml --spawn-count 100 --spawn-rate 100 --max-rps 1000 --request-increase-rate 100
```

### 设置循环次数

除了手动控制压测时长外，HttpRunner v4.0 还支持按照指定的循环次数执行压测，可通过 `--loop-count` 参数进行指定。

需要说明的是，HttpRunner v4.0 执行循环次数的逻辑参考的是 JMeter，所有虚拟用户均会运行指定的循环次数，即当次压测的整体运行次数为：`spawn-count * loop-count`。

执行如下命令，执行完指定循环次数后，HttpRunner 将结束运行并打印整体测试数据。

```bash
$ hrp boom testcase.yml --spawn-count 1000 --spawn-rate 100 --loop-count 10000
```

## 执行性能测试

配置好命令行参数后，即可开始执行性能测试。在性能测试过程中，我们可以通过两种方式查看到性能测试结果数据。

### 命令行终端

性能测试过程中，hrp 每隔 3s 会打印一次性能数据，其中汇总了 3s 内的所有请求情况。结果样式如下：

```text
Current time: 2022/05/16 03:33:26, Users: 1000, State: running, Total RPS: 533.7, Total Average Response Time: 436.8ms, Total Fail Ratio: 0.0%
Accumulated Transactions: 16176 Passed, 3 Failed
+--------------+--------------------+------------+---------+--------+---------+------+-------+--------------+------------+-------------+
|     TYPE     |        NAME        | # REQUESTS | # FAILS | MEDIAN | AVERAGE | MIN  |  MAX  | CONTENT SIZE | # REQS/SEC | # FAILS/SEC |
+--------------+--------------------+------------+---------+--------+---------+------+-------+--------------+------------+-------------+
| request-GET  | get with params    |        799 |       0 |    320 |  432.20 |  273 |  1854 |          266 |     266.33 |        0.00 |
| request-POST | post with params   |        156 |       0 |    280 |  388.44 |  272 |  3633 |          358 |      52.00 |        0.00 |
| request-POST | post with params 2 |        799 |       1 |    400 |  594.02 |  275 | 30000 |          357 |     266.33 |        0.33 |
| transaction  | tran1              |        799 |       0 |    320 |  432.37 |  273 |  1854 |            0 |     266.33 |        0.00 |
| transaction  | Action             |        799 |       1 |   4100 | 4939.17 | 3198 | 35450 |            0 |     266.33 |        0.33 |
+--------------+--------------------+------------+---------+--------+---------+------+-------+--------------+------------+-------------+
```

针对打印的指标说明如下：

| 指标名称 | 指标说明 |
| :--: | :--: |
| Users | 当前虚拟用户数 |
| State | HttpRunner当前运行状态 |
| Total RPS | 总RPS |
| Total Response | 总响应时间 |
| Total Fail Ratio | 总请求错误率 |
| Accumulated Transactions | 事务总通过/失败数，包含Action事务 |
| TYPE | 请求类型 |
| NAME | 请求名称 |
| REQUESTS | 请求数 |
| FAILS | 请求失败数 |
| MEDIAN | 响应时间中位数 |
| AVERAGE | 响应时间平均值 |
| MIN | 响应时间最小值 |
| MAX | 响应时间最大值 |
| CONTENT SIZE | 响应内容大小 |
| REQS/SEC | 每秒请求数 |
| FAILS/SEC | 每秒请求失败数 |

注：表格内皆为统计间隔(3s)内的性能指标数据

完成性能测试时，会打印整体测试数据，结果样式如下：

```text
=========================================== Statistics Summary ==========================================
Current time: 2022/05/16 03:33:38, Users: 1000, Duration: 57s, Accumulated Transactions: 20947 Passed, 3 Failed
+-------+------------+---------+--------+---------+-----+-------+--------------+------------+-------------+
| NAME  | # REQUESTS | # FAILS | MEDIAN | AVERAGE | MIN |  MAX  | CONTENT SIZE | # REQS/SEC | # FAILS/SEC |
+-------+------------+---------+--------+---------+-----+-------+--------------+------------+-------------+
| Total |      31182 |       2 |    300 |  436.70 | 268 | 30003 |          325 |     547.05 |        0.04 |
+-------+------------+---------+--------+---------+-----+-------+--------------+------------+-------------+
```

`Statistics Summary` 展示的是性能测试整个过程中的总体指标数据；指标说明与上面表格一致，只额外增加了一个指标。

| 指标名称 | 指标说明 |
| :--: | :--: |
| Duration | 测试持续时间 |

### 查看性能监控

除了在终端中查看性能测试过程和结果汇总数据之外，我们还可以通过配置 Prometheus + Grafana 看板，实现 Web 化的实时监控指标展示。

效果如下所示：

![](/image/demo-grafana.png)

限于篇幅，关于 Prometheus + Grafana 的性能监控配置方面的内容，将在下一篇文章《如何使用 HttpRunner v4.0 开展性能监控》中详细进行介绍。

## 结语

通过以上示例，我们使用 HttpRunner v4.0 完成了一次性能测试操作实践，相信大家应该对 HttpRunner v4.0 的性能测试使用方法有了一个整体的认识。

欢迎大家多多实践，如果在使用过程中遇到任何问题，欢迎通过各种渠道进行反馈，我们将及时进行跟进和解答，并基于大家的反馈进行迭代优化。

接下来，我们将连载介绍 HttpRunner v4.0 的更多性能测试功能特性，敬请期待。

> 本文作者：[xucong053]，HttpRunner 核心开发者，重点建设了性能测试相关能力<br/>
> HttpRunner 项目官网: https://httprunner.com/<br/>
> 如果 HttpRunner 对你有过帮助，麻烦帮忙给个 ⭐️star⭐️ 鼓励下吧<br/>
> https://github.com/httprunner/httprunner


[HttpRunner v4.0 全新发布]: https://httprunner.com/blog/release-v4/
[Boomer]: https://github.com/myzhan/boomer
[xucong053]: https://github.com/xucong053
