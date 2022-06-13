---
title: 🚀 快速上手
weight: 2
description: 一个实例带你快速上手 QuickRunner
type: docs
---

本文将通过一个简单的示例来展示 QuickRunner 的核心功能使用方法。

## 案例介绍

为了降低案例实践的上手门槛，我们挑选了一个公网可访问的接口服务网站 [http-bin][http-bin] 作为被测对象。`http-bin` 是一个使用 Python + Flask 编写的 HTTP HTTP Request & Response Service。该服务主要用于测试 HTTP 库，你可以向它发送请求，然后它会按照指定的规则将你的请求返回。
httpbin 支持HTTP/HTTPS，可以很好地满足我们的演示需求。

为了更直观，我们将基本的测试流程分为以下几步：

## 测试场景设计

成功的负载测试需要设计一个全面的测试计划。清晰、明确的测试计划可确保您开发的 QuickRunner 场景能够实现您的负载测试目标。

## 测试前准备

#### 工具安装

根据不同的系统选择对应的版本工具进行下载安装。安装成功后，打开工具，界面如下图所示：

<img src="/image/QuickRunner/QuickRunner.png" alt="QuickRunner" width="800">

#### 脚本准备

在测试之前，我们需要对场景脚本进行录制或者脚本编写。

<img src="/image/QuickRunner/ready.png" alt="QuickRunner" width="800">

#### 脚本调试

当脚本设置好后，点击界面右上方的【调试】按钮可对脚本进行调试，以此来检查脚本是否可以跑通。

<img src="/image/QuickRunner/debug.png" alt="QuickRunner" width="800">

#### 压测执行

开启压测之前，您可在【压测配置】页面，设置压测配置，定义运行模式。设置结束后，点击【启动】按钮，开启压测。

<img src="/image/QuickRunner/demo-pressing.png" alt="QuickRunner" width="800">

#### 结果展示

您可在【压测任务】界面查看压测过程中的所有指标数据。

<img src="/image/QuickRunner/demo-result.png" alt="QuickRunner" width="800">


## 总结

到此为止，QuickRunner 的全套基本测试流程就介绍完了，您可以根据本文中的流程，快速开启压测。

当然，QuickRunner 的功能不止于此，如需挖掘 QuickRunner 的更多特性，实现更复杂场景的自动化测试需求，可继续阅读后续文档。


[http-bin]: https://httpbin.org/
