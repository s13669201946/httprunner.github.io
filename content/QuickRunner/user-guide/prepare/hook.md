---
title: "hook机制"
weight: 3
type: docs
---
QuickRunner 提供支持 hook 机制，可以帮助您在发送请求前做一些预处理或在请求完成后做一些后置处理。目前 QuickRunner 支持两种 hook 方式，前置操作主要用于处理接口的前置的准备工作，后置操作主要用于测试后的清理工作，您可以在前置操作或者后置操作里引用您的自定义函数。

<img src="/image/QuickRunner/direction/hook1.jpeg" alt="QuickRunner" width="800">


>温馨提示：QuickRunner 目前的 hook 机制的作用范围只限于在 test 测试用例，它的作用域是当前 test 用例有效。
