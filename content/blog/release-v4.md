---
title: "HttpRunner v4.0 全新发布"
slug: release-v4
date: 2022-04-26
description: 一文了解 HttpRunner v4.0 的前世、今生与未来
---

## v4.0 的诞生背景

HttpRunner 经过近 5 年的迭代，即将进入到 v4.0 版本了。非常欣喜的是，HttpRunner 已经有了较大的用户基数和知名度，在搜索引擎和各种主流技术社区搜索 HttpRunner 都能看到一些用户自发分享的文章，甚至还有培训班以此开设了付费课程，以及有人写书时做了较大篇幅的介绍。这些反馈给了我极大的鼓舞，让我有更大的动力将 HttpRunner 变得更好。

那 HttpRunner v4.0 作为一个全新的大版本，诞生的背景是什么？期望达成的目标是什么呢？

在[我的 2021 年终总结][2021]中有讲到，我所在的团队从去年开始重点投入到 ToB 方向。经过一段时间的探索和思考之后，我们选择以接口性能测试工具作为切入点（产品命名 `QuickRunner`），并计划采用 `Open Core` 的商业模式（核心能力基于开源的 HttpRunner），希望可以通过这种方式更多地触达用户收集反馈，借助开源社区打磨产品。

不过，既有的 HttpRunner 可能还没法直接满足我们的需求。因为我们的核心目标是要做一款性能测试工具，对于发压能力和数据准确度具有非常高的要求；而之前的 HttpRunner 是基于 Python 开发的，性能测试部分使用的是 Locust，其单机发压能力和数据准确性都存在较大的问题。

经过短暂的思考后，我打算采用 Golang 按照 HttpRunner v2 的思路重写一遍，在保持使用方式基本不变的前提下，使功能更丰富、运行更稳定、性能更强、更易部署和使用。同时，为了避免 Golang 版本在迭代初期过程中影响到已有的 Python 版本，我还新建了一个新的项目，命名为 [HttpRunner+]（简称 hrp），在项目结构基本稳定后才合并到 [HttpRunner] 仓库，这也就是我们即将发布的 v4.0 版本。

## v4.0 的核心目标

总的来说，HttpRunner v4.0 最核心的目标是支撑 QuickRunner 成为行业领先的专业级一体化 API 测试解决方案。

具体方面，HttpRunner v4.0 在继承 `v2/v3` 已有功能和保持前向兼容的基础上，重点会在如下几个维度进行提升：

- 简单易用、功能强大：这是 HttpRunner 一直以来从未改变过的目标；在保持简单易用的基础上，v4.0 会重点支持双执行引擎、多网络协议、多编程语言、多测试用途等功能
- 数据精准：v4.0 致力于成为一款专业级的 API 测试工具，不管是单用户的接口请求耗时和网络性能采集，还是高并发下的性能测试指标，测试结果数据精准是底线，必须通过严格的 benchmark 测试
- 海量并发：重点增强压测能力，占用资源更少，发压效率更高，单机轻松支持万级别并发压力，结合分布式稳定支撑百万级真实并发压力测试
- 开源生态：充分与 API 相关工具和标准进行打通，增强数据流通性和二次开发可扩展能力

## v1/v2/v3/v4 版本对比概览

可能你还是会感到有些困惑，HttpRunner v4 相比于之前的 v1/v2/v3/hrp 版本，他们的关系和差异到底是什么？

为了让你感观更加清晰，我整理出了如下表格，详细对比了各个版本间的关键差异点。

| 版本 | v1 | v2 | v3 | HttpRunner+ | v4 |
| :--: | :--: | :--: | :--: | :--: | :--: |
| 发布时间 | 2018.3.7 | 2019.1.1 | 2020.03.10 | 2021.11.18 | 2022.04.29 |
| 开发语言 | Python | Python | Python | Golang | Golang + Python |
| 版本号规范（semver） | ❌ | ✅ | ✅ | ✅ | ✅ |
| 网络协议 | HTTP(S)/1.1 | HTTP(S)/1.1 | HTTP(S)/1.1 | HTTP(S)/1.1 | **多协议** HTTP(S)/HTTP2/WebSocket/_TCP/RPC_ |
| 脚本转换工具 | HAR | HAR | HAR |HAR | HAR/_Postman/Swagger/Curl_ |
| 工程脚⼿架 | ❌ | ✅ | ✅ | ✅ | ✅ |
| 测试⽤例（集）格式 | v1 | v2 | v2 | v2 | v2 |
| 测试⽤例分层机制 | v1 | v2 | v2 | v2 | v2 |
| 脚本格式类型 | YAML/JSON | YAML/JSON | YAML/JSON/**pytest** | YAML/JSON | YAML/JSON/**pytest**/**gotest** |
| 脚本格式校验 | ❌ | [jsonschema] | ❌ | ❌ | _TODO_ |
| 脚本编写语法提示 | ❌ | ❌ | pytest 链式调用 | gotest 链式调用 | gotest 链式调用 + pytest 链式调用 |
| 脚本执行引擎 | Python unittest | Python unittest | Python **pytest** | Go 自研 | Go 自研 + Python **pytest** |
| 插件化语言（debugtalk.xx） | Python | Python | Python | **多语言**（Go/Python） | **多语言**（Go/Python/_Java/etc._） |
| 参数提取机制 | regex + 点分隔符 | jmespath + regex + 点分隔符 | jmespath | jmespath + regex | jmespath + regex |
| skip 机制 | ✅ | ❌ | ❌ | ❌ | _TODO_ |
| 接口测试报告 | html 自研（jinja2） | html 自研（jinja2） | pytest-html/allure | html 自研（Go template） | html 自研（Go template） + _pytest-html/allure_ |
| 性能测试引擎 | Python Locust | Python Locust | Python Locust | Go Boomer | Go Boomer |
| 运行环境依赖 | Python 2.7/3.3+ | Python 2.7/3.5+ | Python 3.7+ pytest | 无需依赖 | Go 引擎无需依赖<br/>pytest 引擎依赖 Python 3.7+ |
| 网络性能采集 | ❌ | ❌ | ❌ | ❌ | ✅ |
| 安装部署方式 | pip | pip | pip | curl/wget | curl/wget |

注：v4 中 _斜体_ 代表当前还未支持，但计划会实现。

从上面的表格可以看出，HttpRunner v4 颇有点**金刚葫芦娃**的意思，囊括了之前所有版本的功能，并且增加了很多新特性。

```
HttpRunner v4 = v2 + v3 + hrp + ...
```

在使用体验和用例格式兼容性方面，v4 也会与之前的 v2/v3/hrp 做到兼容，因此后续 HttpRunner 的维护工作都将转到 v4 版本，之前的版本将不再单独维护。

## v4.0 核心功能介绍

在 v4.0 版本中，HttpRunner 整体完成了重新设计，重点支持了双执行引擎、多网络协议、多编程语言、多测试用途等能力。

这里先只挑选部分进行简单介绍，后续我们会补充完善的用户说明文档。

### 双执行引擎

HttpRunner v4.0 可能给很多用户带来的第一直观印象，就是将项目语言替换为了 Golang。

这种说法其实是不太准确的，确切地说，HttpRunner v4.0 同时采用了 Golang/Python 两种编程语言，底层会有两套相对独立的执行引擎，目标是兼具 Golang 的高性能和 pytest 的丰富生态。

具体实现方面，HttpRunner v4.0 采用 Golang 基本上实现了所有的功能，除了用例执行引擎（hrp run）和压测引擎（hrp boom）外，脚手架工具（hrp startproject）、用例生成工具（hrp har2case）、脚本转换工具（hrp convert）等都优先采用 Golang 进行编写，目的是提升执行效率（毕竟是 Go）和代码质量（静态语言 & 类型系统）。而唯一继续采用 Python 进行编写的则是基于 Python pytest 的接口测试执行引擎（hrp pytest），主要考虑是支持 pytest 的丰富插件生态，并且与 v3 保持兼容。

不过需要特别说明的是，使用 HttpRunner v4.0 并不是必须要具备 Go 语言基础。

HttpRunner 的测试用例脚本支持：

- 文本形态：JSON、YAML
- 代码形态：pytest 和 go test

并且动态运算逻辑（plugin）支持多种编程语言（详见后文），包括 Python。

因此你完全可以选择使用 JSON/YAML 格式编写维护测试用例，插件语言选择 Python，使用方式和体验可以做到基本上和 HttpRunner v2.x 一致。或者你可以选择 pytest 代码形态的脚本，在使用体验上也可以做到和 v3.x 基本一致。

但如果你是期望使用 HttpRunner v4.0 做专业的性能测试，那么最好还是选择使用 golang 编写插件，单机发压性能（QPS）可以达到 2w 以上，这是 Python 插件远远无法达到的。

### 多网络协议

HttpRunner v4.0 已不再局限于 HTTP 协议，当前已成长为一个支持多协议可扩展的测试工具。

截至当前，HttpRunner v4 已经新增支持了 HTTP/2 和 WebSocket 协议，RPC（thrift）协议正在开发中，后续将逐步支持 TCP/UDP/gRPC 等协议类型。

不过，HttpRunner 会继续保持该命名。HTTP 作为最广泛使用的网络协议，可以作为协议的代表，就像 Postman 中的 POST 那样，也是可以讲得通的。

### 多编程语言

针对动态运算逻辑（plugin）部分，HttpRunner v4.0 不再局限于 Python，而是采用了 gRPC 的方案。因此理论上，[gRPC] 支持 10+ 主流编程语言 HttpRunner v4 都可以支持。

当前已支持的插件语言：

- Go
- Python
- Java（计划中，期待贡献）

这部分能力已经单独抽离出了一个基础组件，有兴趣可以详见 [httprunner/funplugin]。

### 多测试用途

HttpRunner 在创建之初就以「能力复用」为目标，期望只需编写维护一套脚本，就可以同时实现接口测试、性能测试、服务可用性监控等能力。

在 v4.0 版本中，HttpRunner 更进一步，将新增支持「网络性能监测」，实现数字体验监测（DEM）的能力。

同时，v4 还对 step 的类型进行了抽象，方便进行扩展。

基于该机制，我们可以扩展支持新的网络协议类型，也可以支持新的测试类型，例如 SQL 操作或 UI 自动化。甚至我们还可以在一个测试用例中混合调用多种不同的 Step 类型，例如实现 HTTP/RPC/SQL/UI 混合场景。

是不是很有想象力空间？

## 开源项目运营

开源项目不仅仅是将代码进行公开，同时还需要建立好用户社区，帮助并引导更多的人加入进来。而在这方面 HttpRunner 一直做得都不大好，接下来必须得加强开源社区运营的投入。

从 v4.0 版本开始，HttpRunner 期望从以下几个方面进行改进。

### 核心开发团队

截至当前，HttpRunner 已经有 21 位[开发者]贡献过代码。虽然看上去还不错，但之前主要的开发维护工作基本上都还是我一个人，这对项目的健康度是非常不利的。

庆幸的是，在 QuickRunner 项目立项之初，我们团队又投入了 2 位同学到 HttpRunner 的开发工作；同时在字节内部，其它部门另一位同样在重度使用 HttpRunner 的同学也加入了开源共建。

因此，HttpRunner 当前有了 4 位真正意义上的核心开发者：

- [debugtalk]：负责项目的整体架构和规划；哪里有砖哪里搬
- [xucong053]：贡献了多机负载分布式压测能力、Prometheus 性能数据采集能力、参数化数据驱动、html 报告等
- [bbx-winner]：贡献了 HTTP2/WebSocket 网络协议、benchmark 数据准确性测试等
- [billduan]：重点投入 pytest 引擎部分，贡献了 thrift RPC 协议、集成 SQL 调用等

在这里也非常欢迎更多的朋友参与进来，特别是在工作中重度使用 HttpRunner 的朋友，「独行者速，众行者远」。

### 用户调研问卷

为了更好地收集 HttpRunner 用户反馈，明确需求迭代路径，我们从今年开始尝试了用户调研问卷的形式。

- [HttpRunner 用户调研问卷][suvey1] => [HttpRunner 首轮用户调研报告（2022.02）][user-survey-report]
- [HttpRunner 增值服务调研问卷][suvey2]
- [QuickRunner 用户需求调研][suvey3]

通过调研问卷，我收集到了非常多宝贵的反馈和建议，这对 HttpRunner 的发展起到了非常重要的指引作用。

在此非常感谢大家的积极参与，也希望大家后续可以通过这个途径给予 HttpRunner 更多的反馈（共性需求会高优实现哦~）。

### Issue 管理 & 版本规划

GitHub [Issues] 是 HttpRunner 的项目需求管理工具，也是 HttpRunner 用户和开发者的主要交流渠道。

大家在使用 HttpRunner 的过程遇到问题都可以通过 issue 反馈（建议先搜下是否有重复问题），我们通过其它渠道收集到的问题也会汇总到这里。

同时为了更高效地管理 issues，HttpRunner 设置了多个维度的标签（Labels），每个 issue 会采用 1~N 个标签进行组合管理。具体可阅读 HttpRunner 的 [Issue 管理机制]。

另一方面，HttpRunner 从 v4.0 开始要做版本规划了。具体形式采用的是 GitHub 的里程碑（milestone）功能，每个版本会设定为一个里程碑，例如 v4.1、v4.2。然后我们会把所有的 issue（包括新功能、优化、bugfix 等）规划到具体版本中，并在 GitHub 的 [Projects] 中进行管理。

如果大家想了解 HttpRunner 后续的版本规划，可以到 [Projects] 中进行查看。当然啦，需求规划后也会动态进行调整，如果你发现你想要的功能排期比较靠后，也可以多多进行反馈。还是那句话，共性需求会高优实现的。

### 用户使用文档

在 [HttpRunner 首轮用户调研报告（2022.02）][user-survey-report] 中，用户使用文档相关的吐槽在所有问题中排名首位，远超其它问题。

这个问题的确非常严重，我们在 v4.0 中必须得改！

当前已经明确的改进措施有三条。

- 用户文档统一采用中文编写，确保清晰易懂。
- 梳理了一份文档计划，已经分工到核心开发者，后续会将文档编写安排到工作排期中。
- 除了用户使用文档，我们还会针对大的功能特性通过博客介绍其原理机制，期待跟大家有更多的交流互动。

### 用户交流社区

HttpRunner 的交流途径挺多的，当前重点维护的包括：

- [TesterHome 社区]：开设了 HttpRunner 主题板块，这也是 HttpRunner 项目的发源地
- [微信公众号]：不定期推送 HttpRunner 重点资讯（搜索 HttpRunner 进行关注）
- [GitHub Discussions]：GitHub 上开设的 discussions 社区板块（公告、想法、答疑、分享）
- 微信交流群：为了保障交流质量特地设置了一些门槛，需要认真填写一份[调研问卷]

如果上面的途径都无法满足你的需求，你也可以加我个人微信（leolee023），但需要在加好友的时候做下稍微详细点的自我介绍（至少包含姓名和所在公司，最好是再注明下期望交流的内容）。

## 总结

以上便是 HttpRunner v4.0 发布将带来的主要变化。在今年的 MTSC 大会的开源专场，我也会对 HttpRunner v4.0 的设计思路和核心原理进行分享，到时候可以线下当面做更多的交流。

最后再做个小预告，我们团队基于 HttpRunner 研发的一体化 API 测试工具 QuickRunner 已经完成了初版迭代，很快就要面向社区开放体验了，感兴趣的朋友可以先关注下。

最后的最后，开源不易，要是 HttpRunner 对你有过帮助，麻烦帮忙给个 ⭐️star⭐️ 鼓励下吧。

https://github.com/httprunner/httprunner

![](/image/star-history.png)

[jsonschema]: https://github.com/python-jsonschema/jsonschema
[2021]: https://debugtalk.com/post/my-2021-summary/
[HttpRunner+]: https://github.com/httprunner/hrp
[HttpRunner]: https://github.com/httprunner/httprunner
[httprunner/funplugin]: https://github.com/httprunner/funplugin
[gRPC]: https://grpc.io/docs/languages/
[开发者]: https://github.com/httprunner/httprunner/graphs/contributors
[debugtalk]: https://github.com/debugtalk
[xucong053]: https://github.com/xucong053
[bbx-winner]: https://github.com/bbx-winner
[billduan]: https://github.com/billduan
[suvey1]: https://wj.qq.com/s2/9699514/0d19/
[suvey2]: https://wj.qq.com/s2/9708369/b6b2
[suvey3]: https://wj.qq.com/s2/10035362/b78e
[user-survey-report]: /blog/user-survey-report
[Issues]: https://github.com/httprunner/httprunner/issues
[Issue 管理机制]: /docs/contribution-guidelines/issues/
[Projects]: https://github.com/orgs/httprunner/projects/1
[TesterHome 社区]: https://testerhome.com/topics/node138
[GitHub Discussions]: https://github.com/httprunner/httprunner/discussions
[微信公众号]: https://httprunner.com/image/qrcode.jpg
[调研问卷]: /blog/user-survey/
