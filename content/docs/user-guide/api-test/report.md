---
title: 测试报告
weight: 3
description: 生成 HTML 接口测试报告
---

使用 `hrp run` 执行接口测试时，我们可以通过添加启动参数 `--gen-html-report` 在当前项目根目录的 reports 中生成一份 HTML 格式的测试报告。测试报告的输出名称为：report-unix时间戳。

## 报告样式

报告样式如下所示，与 HttpRunner v2.x 版本保持一致。

![img.png](/image/report-1.png)

## 报告说明

在 Summary 中，会罗列本次测试的整体信息，包括测试开始时间、总运行时长、运行的 Go 版本和系统环境、运行结果统计数据。

在 Details 中，会详细展示每一条测试用例的运行结果。

其中，每一条测试用例对应一个 log 按钮，点击 log 按钮，会在弹出框中展示该用例执行的详细数据，包括请求的 headers 和 body、响应的 headers 和 body、校验结果、响应、响应耗时（elapsed）等信息。具体细节如下图所示：

![img_1.png](/image/report-2.png)
![img_2.png](/image/report-3.png)

此外，若测试用例运行不成功（error），则在该测试用例的 Detail 中会出现 traceback 按钮，点击该按钮后，会在弹出框中展示失败原因。

