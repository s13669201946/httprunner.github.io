---
title: "整体概览"
weight: 11
description: HttpRunner 的概述介绍、设计理念、特性特性
---

`HttpRunner` 是一个开源的 API 测试工具，支持 HTTP(S)/HTTP2/WebSocket/RPC 等网络协议，涵盖接口测试、性能测试、数字体验监测等测试类型。简单易用，功能强大，具有丰富的插件化机制和高度的可扩展能力。

![flow chart](/image/hrp-flow.jpg)

## 设计理念

相比于其它 API 测试工具，HttpRunner 最大的不同在于设计理念。

- 约定大于配置：测试用例是标准结构化的，格式统一，方便协作和维护
- 标准开放：基于开放的标准，支持与 [HAR]/Postman/Swagger/Curl/JMeter 等工具对接，轻松实现用例生成和转换
- 一次投入多维复用：一套脚本可同时支持接口自动化测试、性能测试、数字体验监测等多种 API 测试需求
- 融入最佳工程实践：不仅仅是一款测试工具，在功能中融入最佳工程实践，实现面向网络协议的一站式测试解决方案

## 核心特性

- 网络协议：完整支持 HTTP(S)/1.1 和 HTTP/2，可扩展支持 WebSocket/TCP/RPC 等更多协议
- 多格式可选：测试用例支持 YAML/JSON/go test/pytest 格式，并且支持格式互相转换
- 双执行引擎：同时支持 golang/python 两个执行引擎，兼具 go 的高性能和 [pytest] 的丰富生态
- 录制 & 生成：可使用 [HAR]/Postman/Swagger/curl 等生成测试用例；基于链式调用的方法提示也可快速编写测试用例
- 复杂场景：基于 variables/extract/validate/hooks 机制可以方便地创建任意复杂的测试场景
- 插件化机制：内置丰富的函数库，同时可以基于主流编程语言（go/python/java）编写自定义函数轻松实现更多能力
- 性能测试：无需额外工作即可实现压力测试；单机可轻松支撑 `1w+` VUM，结合分布式负载能力可实现海量发压
- 网络性能采集：在场景化接口测试的基础上，可额外采集网络链路性能指标（DNS 解析、TCP 连接、SSL 握手、网络传输等）
- 一键部署：采用二进制命令行工具分发，无需环境依赖，一条命令即可在 macOS/Linux/Windows 快速完成安装部署

## 用户声音

基于 252 份调研问卷的统计结果，HttpRunner 用户的整体满意度评分 `4.3/5`，最喜欢的特性包括：

- 简单易用：测试用例支持 YAML/JSON 标准化格式，可通过录制的方式快速生成用例，上手简单，使用方便
- 功能强大：支持灵活的自定义函数和 hook 机制，参数变量、数据驱动、结果断言等机制一应俱全，轻松适应各种复杂场景
- 设计理念：测试用例组织支持分层设计，格式统一，易于实现测试用例的维护和复用

更多内容详见 [HttpRunner 首轮用户调研报告（2022.02）](/blog/user-survey-report)

[HAR]: https://en.wikipedia.org/wiki/HAR_(file_format)
[pytest]: https://docs.pytest.org/
