---
title: "常见问题"
weight: 13
description: 关于 HttpRunner 的常见问题（持续补充中）
---

## HttpRunner 名字的由来？

项目命名为 HttpRunner 主要有以下几点缘由：

- 项目最初的目标是针对 HTTP 协议实现一套接口和性能测试工具
- Runner 借鉴了 LoadRunner，这个词用在自动化测试和性能测试上都比较自然
- HttpRunner 这个组合词还未有人使用，GitHub/PyPI/域名/微信公众号 都是可用状态

这里也着重说明两点：

- 虽然 HTTP 的正确写法是全大写，但 `HttpRunner` 是一个组合后的新专有名词，可以不受此限制；`HTTPRunner` 的写法是错误的
- HttpRunner 已不再局限于 HTTP 协议，当前已成长为一个支持`多协议可扩展`的测试工具；HTTP 作为最广泛使用的网络协议，可以作为协议的代表，就像 Postman 中的 POST 那样

## HttpRunner 支持的语言有哪些？

HttpRunner 的测试用例脚本支持：

- 文本形态：JSON、YAML
- 代码形态：pytest 和 go test

针对动态运算逻辑（plugin）部分，HttpRunner 支持 gRPC 的方案，因此理论上，gRPC 支持的语言 HttpRunner 都可以支持。

当前已支持的插件语言：

- Go
- Python
- Java（计划中）

这部分能力已经单独抽离出了一个基础组件，详见 [httprunner/funplugin]。


## HttpRunner logo 的含义

HttpRunner 的 Logo 如下所示：

<img src="/image/logo.png" alt="HttpRunner Logo" width="200">

对于 logo 设计的解释，主要有如下两点：

- 中间是个拼图（puzzle pieces），形似 H 字母，恰好是 HttpRunner 的首字母
- 拼图的寓意，对应了 HttpRunner 的设计理念：HttpRunner 本身作为一个基础框架，可以组装形成各种类型的测试工具和平台；而在 HttpRunner 内部，也是充分解耦的各个模块组装在一起形成的

## HttpRunner 的开源协议？

HttpRunner 采用了非常宽松的开源协议 Apache-2.0，商业友好，可以永久免费使用无任何限制。

## Go 版本与 Python 版本的差异？

## 使用 HttpRunner v4.0 必须要学习 Go 语言么？

先说结论，HttpRunner v4.0 不需要用户具有 Golang 基础。

HttpRunner v4.0 支持 JSON/YAML 格式的测试用例，动态运算逻辑（plugin）支持多种编程语言（包括 Python），因此使用的方式和体验可以做到基本和 v2.x 一致。

同时，HttpRunner v4.0 具有双引擎，完整支持 pytest 格式的脚本，因此在使用体验上也可以做到和 v3.x 基本一致。

但如果你是期望使用 HttpRunner v4.0 做专业的性能测试，那么最好还是选择使用 golang 编写插件，单机发压性能（QPS）可以达到 2w 以上，这是 Python 插件远远无法达到的。


[httprunner/funplugin]: https://github.com/httprunner/funplugin