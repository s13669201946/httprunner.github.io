---
title: 用例增强
weight: 1
description: 在性能测试脚本中支持添加事务、集合点、思考时间等机制
---

在性能测试之前，我们需要先准备好性能测试用例。

在 HttpRunner 中，得益于「一体化」的特性优势，我们可以在无需对已有接口测试用例做任何修改的情况下，将测试命令从 `hrp run` 改为 `hrp boom`，即可启动性能测试。具体的参数配置详见[参数介绍]。

需要说明的是，性能测试的测试用例格式只能选择 `YAML/JSON/GoTest`；PyTest 格式底层基于 Python 执行引擎，考虑到性能和数据准确性问题，已不再支持性能测试。

另外，HttpRunner v4.0 对标 LoadRunner 新增实现了丰富的性能测试机制，包括「事务」、「集合点」、「思考时间」等，可以在已有的接口测试用例基础上添加特定的步骤进行用例增强。

## 事务（transaction）

> 事务可以将多个接口的测试结果进行聚合统计，性能指标更加贴合真实业务场景。

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

## 集合点（rendezvous）

> 集合点可以确保指定的虚拟用户在同一时刻发起请求，实现类似「秒杀」的真实业务场景。

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

可以看出，HttpRunner v4.0 的「集合点」机制也跟 LoadRunner 保持了一致，功能特性和参数配置方法完全相同。

## 思考时间（think_time）

> 思考时间可以模拟用户在不同操作间的停顿时间，最大程度还原用户真实的操作行为。

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

- time: 不同测试步间的思考时间，单位为秒（second）

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

在该示例中，思考时间策略为随机比例（`random_percentage`），介于 100% ~ 150% 之间；limit 设置为 4s。经过换算，测试步骤中的 `think time 1` 的最终值在 [2, 3]s 之间，`think time 2` 的最终值在 [3, 4]s 之间。

## 用例格式示例

包含「事务」、「集合点」、「思考时间」机制的性能测试用例示例如下。

YAML 用例格式：

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

对应的 GoTest 用例格式：

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

[参数介绍]: /docs/user-guide/load-test/boom/
