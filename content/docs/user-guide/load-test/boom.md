---
title: hrp boom
weight: 2
description: 通过 hrp boom 命令行工具启动性能测试
---

准备好性能测试用例后，我们就可以开始使用 `hrp boom` 执行性能测试了。

## 性能测试参数配置

当前 HttpRunner v4.0 支持多种性能测试策略，我们可以在命令行参数中进行指定。

关于性能测试的参数信息，可以通过 `hrp boom -h` 进行查看。

```text
$ hrp boom -h
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

如果你之前使用过 Locust/Boomer，你会发现参数名称很眼熟。是的，我们直接复用了 Locust/Boomer 的参数名称，虽然底层实现进行了比较大的改造，但用法基本保持了一致，你可以对比查看下 [Boomer 的参数文档]。

参数看上去比较多，接下来对其按照用途进行分组介绍。

### 设置并发用户数

性能测试需要指定并发用户数，同时在很多时候我们需要按照阶梯加压的方式初始化并发用户。这里就需要使用到 `--spawn-count` 和 `--spawn-rate` 两个参数了。

- spawn-count：表示我们期望达到的最大并发用户数
- spawn-rate：表示达到期望最大并发前每秒增加的并发用户数

具体示例如下所示：

```bash
$ hrp boom testcase.yml --spawn-count 1000 --spawn-rate 100
```

在该示例中，我们指定了 1000 并发用户，按照每秒 100 个的速率初始化用户，预计在 10s 后可完成初始化。

在这种模式下，性能测试不会自动结束（除非指定循环次数或者压测时长）；当我们期望结束压测时，可以通过 kill 命令杀死进程，或者 `Ctrl+C` 来结束性能测试，结束时终端会打印整体测试数据。

### 设置 RateLimiter

有时我们需要在性能测试时对发压流量进行限流，例如期望限制最大的 RPS，或者在初始化并发用户时限定增加数率。这就需要使用到 `--max-rps` 和 `--request-increase-rate`。

- max-rps：限制 hrp 发压的最大 rps，默认不限制
- request-increase-rate：限制 hrp 每秒的请求增加速率，默认不开启，支持两种格式："1"、"1/1s"

具体示例如下所示：

```bash
$ hrp boom testcase.yml --spawn-count 100 --spawn-rate 100 --max-rps 1000 --request-increase-rate 100
```

在该示例中，我们指定了 100 并发用户，按照每秒 100 个的速率初始化用户，预计在 1s 后可完成初始化。同时，我们限制在整个压测过程中，hrp 最高只能发压 1000 RPS，同时限定每秒增加的速率为 100 RPS。

### 设置循环次数

除了手动控制压测时长外，HttpRunner v4.0 还支持按照指定的循环次数执行压测，可通过 `--loop-count` 参数进行指定。

需要说明的是，HttpRunner v4.0 执行循环次数的逻辑参考的是 JMeter，所有虚拟用户均会运行指定的循环次数，即当次压测的整体运行次数为：`spawn-count * loop-count`。

执行如下命令，执行完指定循环次数后，HttpRunner 将结束运行并打印整体测试数据。

```bash
$ hrp boom testcase.yml --spawn-count 100 --spawn-rate 10 --loop-count 1000
```

在该示例中，我们指定了 100 个并发用户，每个用户循序运行 1000 次，预计总共运行 100*1000=10w 次。

### 设置运行时长（TODO）

### 关闭终端打印

如果你想在终端中关闭打印信息，可以在命令行中添加一个 `--disable-console-output` 参数。

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

关于 Prometheus + Grafana 的性能监控配置方面的内容，详见[监控配置]。


[hrp_boom]: https://github.com/httprunner/httprunner/blob/master/docs/cmd/hrp_boom.md
[Boomer 的参数文档]: https://boomer.readthedocs.io/en/latest/options.html
[监控配置]: /docs/user-guide/load-test/monitor/
