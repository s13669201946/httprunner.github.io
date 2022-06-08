---
title: 录制生成
weight: 1
description: 基于 HAR 生成测试用例
---

## 什么是 HAR 格式

> 以下内容来自维基百科关于 [HAR](https://zh.wikipedia.org/wiki/.har) 格式的介绍 

HTTP 存档格式（HTTP Archive format，简称 HAR）是一种 JSON 格式的存档文件格式，多用于记录网页浏览器与网站的交互过程。文件扩展名通常为 .har。

HAR 格式的规范定义了一个 HTTP 事务的存档格式，可用于网页浏览器导出加载网页时的详细性能数据。此格式的规范由万维网联盟（W3C）的 Web 性能工作组制作，截至2016年3月是一份规范草案。

## 录制 HAR 文件

采用 Charles 等抓包工具，以及 Chrome 等浏览器均可以导出 HAR 格式的文件，具体方式可以参考如下流程：

### Charles 录制 HAR 文件

Charles 录制 HAR 分为如下两步：
- 步骤一：在抓取到的会话（Session）上右击选择 `Export Session...`

![Charles 录制 HAR 文件步骤1](/image/charles-record-har-1.png)

- 步骤二：导出格式选择 `HTTP Archive (.har)` 来导出为 HAR 文件

![Charles 录制 HAR 文件步骤2](/image/charles-record-har-2.png)

### Chrome 录制 HAR 文件

Chrome 录制 HAR 文件可以通过如下两种方法：

- 方法一：点击按钮导出 HAR 文件
- 方法二：在请求列表空白处右击，选择`以 HAR 格式保存所有内容`

![Chrome 录制 HAR 文件](/image/chrome-record-har.png)

在录制请求并导出 HAR 文件之后，就可以进行测试用例的生成了，这里首先执行 `hrp har2case -h` 来查看指令的帮助信息。

```text
$ hrp har2case -h
convert HAR to json/yaml testcase files

Usage:
  hrp har2case $har_path... [flags]

Flags:
  -h, --help                help for har2case
  -d, --output-dir string   specify output directory, default to the same dir with har file
  -p, --profile string      specify profile path to override headers and cookies
  -j, --to-json             convert to JSON format (default true)
  -y, --to-yaml             convert to YAML format

Global Flags:
      --log-json           set log to json format
  -l, --log-level string   set log level (default "INFO")
```

由于 `hrp convert` 指令在选项与功能上都兼容和涵盖了 `hrp har2case`，因此选项的详细说明与使用示例将在[转换生成]章节中进行介绍，因此本节仅给出一个简单的示例如下：

```shell
# 将输入的 demo.har 转换为 yaml 测试用例 demo_test.yaml
$ hrp har2case demo.har --to-yaml
```

[HAR]: https://zh.wikipedia.org/wiki/.har
[转换生成]: https://httprunner.com/docs/user-guide/gen-tests/convert/
