---
title: "整体概览"
weight: 11
description: HttpRunner+ 是什么？
---

`HttpRunner+`（简称 `hrp`）是一个开源的网络协议测试工具，基于录制生成的一套脚本，可同时满足接口自动化测试、性能测试、数字体验监测、测试数据生成等多维度需求。简单易用、功能强大、数据精准永远是 `hrp` 最核心的目标。

当前 `hrp` 完整支持 HTTP/1.1，后续将逐步扩展支持更多网络协议。

## 设计理念

相比于 JMeter 等其它测试工具，hrp 最大的不同在于设计理念。

- 约定大于配置：测试用例是标准结构化的，格式统一，方便协作和维护；同时可与海量工具对接，易于实现用例生成和转换
- 一次投入多处复用：回归协议测试的本质，一套脚本同时实现接口自动化、性能测试、数字体验监测、测试数据生成等多种需求
- 融入最佳实践：不仅仅是一款测试工具，在功能中融入最佳工程实践，实现面向网络协议的一站式测试解决方案

## 流程图

![flow chart](/image/flow.jpg)

## 核心特性

- 网络协议：完整支持 HTTP/1.1，后续将逐步扩展支持 HTTP/2, TCP, RPC, WebSocket 等更多协议
- 多格式可选：测试用例支持使用 YAML/JSON/Golang/Python 格式，并且它们可互相转换
- 录制 & 生成：可使用 HAR/Postman/Swagger 生成测试用例；基于链式调用的方法提示也可快速编写测试用例
- 复杂场景：基于 variables/extract/validate/hooks 机制可以方便地创建任意复杂的测试场景
- 性能测试：无需额外工作即可实现压力测试；单机可轻松支撑 1w+ QPS，结合分布式负载能力可实现海量发压
- 网络性能采集：在场景化接口测试的基础上，可额外采集网络链路性能指标（DNS 解析、TCP 连接、SSL 握手、网络传输等）
- 命令行工具 / 库函数：同时支持二进制命令行工具和 Golang/Python 库函数引用两种使用方式
- 插件化机制：内置丰富的函数库，同时可以基于自定义插件轻松实现更多能力
- 高度可扩展性：支持二次开发，可轻松实现接口测试、压力测试、测试数据服务等工具平台
- 开源免费：采用开源协议 Apache-2.0，免费使用无任何限制

## 用户声音

通过调研问卷统计分析，HttpRunner 用户最喜欢的特性包括：

- 简单易用：测试用例支持 YAML/JSON 标准化格式，可通过录制的方式快速生成用例，上手简单，使用方便
- 功能强大：支持灵活的自定义函数和 hook 机制，参数变量、数据驱动、结果断言等机制一应俱全，轻松适应各种复杂场景
- 设计理念：测试用例组织支持分层设计，格式统一，易于实现测试用例的维护和复用

更多内容详见 [HttpRunner 首轮用户调研报告（2022.02）](/blog/user-survey-report)


[HttpRunner]: https://github.com/httprunner/httprunner
[Boomer]: https://github.com/myzhan/boomer
[locust]: https://github.com/locustio/locust
[jmespath]: https://jmespath.org/
[allure]: https://docs.qameta.io/allure/
[HAR]: http://httparchive.org/
[plugin]: https://pkg.go.dev/plugin
[demo.json]: https://github.com/httprunner/hrp/blob/main/examples/demo.json
[examples]: https://github.com/httprunner/hrp/blob/main/examples/
