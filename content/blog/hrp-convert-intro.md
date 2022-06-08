---
title: HttpRunner v4.0 用例转换功能介绍
slug: hrp-convert-intro
date: 2022-06-02
---

## 引言

使用过 HttpRunner v3.0 的用户应该对 `httprunner har2case` 和 `httprunner make` 指令不陌生，这两个指令可以非常方便地进行测试用例的生成与转换，前者用于将 HAR 包转换为 JSON/YAML/pytest 测试用例，后者则用于将 JSON/YAML 测试用例转换为 pytest 形态。

在 v4.0 版本中，HttpRunner 将实现一套更为灵活的测试用例转换机制，并且将所有的转换功能都集中在 `hrp convert` 一个指令中。用户可以通过指定选项将将输入转换为各种类型的测试用例，HttpRunner 的转换功能将不仅仅局限于支持 Postman、Swagger 等工具导出的工程文件到 HttpRunner 测试用例的转换，还会支持 curl、Apache ab 等指令到测试用例的转换，甚至是会支持测试用例的不同形态之间的相互转换。

本文将介绍 HttpRunner v4.0 用例转换功能的基本使用方式以及注意事项，从而可以方便用户快速上手。此外，为了帮助大家更好地理解用例转换的原理，本文还会介绍 `hrp convert` 指令转换过程的具体流程，最后会介绍 `hrp convert` 指令当前的开发进度。

## 快速上手

通过执行 `hrp convert -h` 可以查询该指令的功能简介、用法和各个选项的介绍：

```text
$ hrp convert -h
convert to JSON/YAML/gotest/pytest testcases

Usage:
  hrp convert $path... [flags]

Flags:
  -h, --help                help for convert
  -d, --output-dir string   specify output directory, default to the same dir with har file
  -p, --profile string      specify profile path to override headers (except for auto-generated headers) and cookies
      --to-gotest           convert to gotest scripts (TODO)
      --to-json             convert to JSON scripts (default true)
      --to-pytest           convert to pytest scripts
      --to-yaml             convert to YAML scripts

Global Flags:
      --log-json           set log to json format
  -l, --log-level string   set log level (default "INFO")
```

`hrp convert` 各个选项的详细说明和使用示例如下：

### 指定输出类型

`--to-json/--to-yaml/--to-gotest/--to-pytest` 用于将输入转化为对应目标形态的测试用例，四个选项中最多只能指定一个。

例如，将输入的 HAR 文件转换为 pytest 测试用例：

```shell
# 将输入的 demo.har 转换为 pytest 测试用例 demo_test.py
$ hrp convert demo.har --to-pytest
```

如果不指定该选项，则默认会将输入转换为 JSON 形态的测试用例，例如：

```shell
# 默认会将输入的 demo.har 转换为 JSON 测试用例 demo_test.json
$ hrp convert demo.har
```

### 指定输出目录

`--output-dir` 后接测试用例的期望输出目录的路径，用于将转换生成的测试用例输出到对应的文件夹，需要注意该路径必须是存在且合法的，例如：

```shell
# 创建输出目录
$ mkdir -p testcase/from/postman

# 将输入的 postman_collection.json 转换为 YAML 测试用例 postman_collection_test.yaml，并导出到 testcase/from/postman 目录下
$ hrp convert postman_collection.json --to-yaml --output-dir testcase/from/postman
```

### 指定全局修改配置文件

`--profile` 后接 profile 配置文件的路径，该文件的后缀可以为 .json/.yaml/.yml，其作用是在转换过程中对测试用例的各个步骤进行全局修改，目前支持修改输入中的 Headers 和 Cookies 信息，并且支持替换（不存在则会创建）以及覆盖两种修改模式，下面给出这两种修改模式的 profile 配置文件示例：

- profile.yaml：根据 profile 替换指定的 Headers 和 Cookies 信息

```yaml
headers:
    Header1: "this header will be created or updated"
cookies:
    Cookie1: "this cookie will be created or updated"
```

- profile_override.yaml：根据 profile 覆盖原有的 Headers 和 Cookies 信息

```yaml
override: true
headers:
    Header1: "all original headers will be overridden"
cookies:
    Cookie1: "all original cookies will be overridden"
```

创建了以上的两个 profile 配置文件后，我们可以使用 `--profile` 选项指定配置文件来进行全局修改，例如：

```shell
# 将输入的 demo.har 转化为 json 测试用例 demo_test.json，并进行全局替换
$ hrp convert demo.har --profile profile.yaml

# 将输入的 demo.har 转化为 pytest 测试用例 demo_test.py，并进行全局覆盖
$ hrp convert demo.har --to-pytest --profile profile_override.yaml
```

## 注意事项

1. 输出的测试用例文件名格式为 `输入文件名称（不带后缀）` + `_test` + `.json/.yaml/.go/.py 后缀`，如果该文件已经存在则会进行覆盖
2. `hrp convert` 可以自动识别输入类型，因此不需要通过选项来手动指定输入类型，如遇到输入类型不支持、加载失败或转换失败的情况，则会输出错误日志并跳过，不会影响其他转换过程的正常进行
3. 在 profile 文件中，指定 `override` 字段为 `false/true` 可以选择全局修改模式为替换/覆盖。需要注意的是，如果不指定该字段则默认的全局修改模式为替换模式
4. 输入为 JSON/YAML 测试用例时，向前兼容 HttpRunner v2.0/v3.0 版本的测试用例，输出则统一采用 HttpRunner v4.0 版本的风格
5. 在进行测试用例的不同形态之间的相互转换时，如果输入测试用例的类型与输出的类型相同，所指定的 `--output-dir` 和 `--profile` 选项仍然生效

## 转换流程图

为了帮助大家更好地理解 `hrp convert` 进行用例转换的原理，笔者将外部文件或指令到 JSON/YAML 测试用例以及代码形态测试用例转换的「三部曲」总结为了下图，虽然同属于测试用例，但是 JSON/YAML 与代码形态是有较大区别的，因此分开进行讨论。下面我们围绕该图，详细介绍一下用例转换的流程。

![hrp convert 转换流程图](/image/convert-steps.svg)

### 外部文件或指令

第一部分紫色背景的是外部文件或指令，输入的文件或指令的类型会被 HttpRunner 自动识别，而不需要用户手动指定。

自动识别的过程又可以分为两步：

首先，是输入的类型判断，程序会根据文件的拓展名或指令的头部来进行输入类型的判断，例如，一个后缀为 .har 的文件只可能是 HTTP 存档格式文件，而一个后缀为 .json 的文件则有可能为 Postman 文件、Swagger 文件或 JSON 测试用例中的一种，而像 `"curl -s https://www.example.com"` 这样字符串作为输入则显然只可能为一个 curl 指令。

其次，是输入的加载，程序会在判断输入类型之后尝试对输入的内容进行加载并构建结构体变量，例如输入为 HAR 文件时，程序会加载该文件，并根据 HTTP 存档格式的规则来进行反序列化以及构建结构体变量，如果加载过程中发现输入的 HAR 文件中存在语法错误，或者得到的结构体变量为空，则会判断为加载失败，如果输入为 JSON 文件时，则上述的加载过程最多可能会重复三次。

同样地，当输入类型为指令时，同样根据该指令的使用规则来构建结构体变量，如果出现选项指定错误，同样会判断为加载失败。输入类型不支持、加载失败都会输出错误日志并跳过，不会影响其他转换过程的正常进行，至此，输入外部文件或指令的类型判断与加载完成。

### JSON/YAML 测试用例

第二部分绿色背景的是 JSON/YAML 测试用例，这两种形态的测试用例具有良好的结构化特性。用户在掌握了用例编写的基本规则后，即使不懂代码或者懂少量代码，也可以快速构建易于平台化管理和维护的测试用例。并且 HttpRunner 底层支持的两种执行引擎是可以使用同一套 JSON/YAML 测试用例的（历史原因导致的部分不兼容问题目前也在努力完善中），兼具 Golang 的高性能和 pytest 的丰富生态，真正做到一套测试用例兼顾接口和性能测试。

从外部文件或指令到 JSON/YAML 测试用例的转换过程较为简单，由于上一步中已经得到了外部文件或指令对应的结构体变量，直接按照 HttpRunner 的规则来构建测试用例对应的结构体变量，并进行一次序列化即可导出 JSON/YAML 文件，而对于 JSON/YAML 测试用例之间的互相转换，同样是借助了中间态的测试用例结构体变量。最后，需要注意的是，指定输出目录以及指定全局修改配置文件的功能同样也是在这一步中实现的。

### pytest/gotest 测试用例

第三部分蓝色背景的是代码形态测试用例，具体分为 gotest 和 pytest 两种形态的测试用例。代码形态测试用例的适用场景是使用 SDK 来进行接入以及满足部分用户的调试和二次开发需求，通过`链式调用`的方式，用户可以在 IDE 中根据函数提示来为测试用例添加更多细节，以及实现一些 JSON/YAML 测试用例无法达到的效果。

这一部分转换能力目前的实现方式是复用 HttpRunner Python 版本 `httprunner make` 的能力，后续会考虑统一采用 Golang 模版引擎的方式来实现中间态的测试用例结构体变量到代码形态测试用例的转换，减少底层在虚拟环境中调用 HttpRunner Python 版本进行用例转换所带来的开销，提升用例转换效率。

最后，对于从代码形态测试用例到 JSON/YAML 测试用例的逆向转换过程，以及 pytest/gotest 测试用例之间的相互转换，目前还处于方案阶段，拟采用类似模版逆向解析的技术或者采用在代码形态测试用例的尾部增加功能函数并导出测试用例的方案来实现。

综上所述，`hrp convert` 可以理解为一个支持多类输入，四类输出的模型。该模型在转换过程中又可以分为三个步骤，跨步骤的转换过程尽量做到解耦合来提高转换效率，最后，`hrp convert` 的核心部分借助了 Golang 的接口机制，从而可以实现更好的可扩展性。

## 开发进度

看了上面的介绍，大家是不是对 HttpRunner v4.0 版本的用例转换功能非常期待呢？不过当前的开发工作其实只完成了 1/3，我们会在后续的迭代开发中使用更多的用例转换功能，具体的开发进度如下表。最后，也欢迎有能力的小伙伴可以参与到开源项目中，贡献更多的用例转换功能，将 `hrp convert` 打造为更加实用的「瑞士军刀」。

| from \ to | JSON | YAML | GoTest | PyTest |
|:---------:|:----:|:----:|:------:|:------:|
|    HAR    |  ✅   |  ✅   |   ❌    |   ✅    |
|  Postman  |  ✅   |  ✅   |   ❌    |   ✅    |
|  JMeter   |  ❌   |  ❌   |   ❌    |   ❌    |
|  Swagger  |  ❌   |  ❌   |   ❌    |   ❌    |
|   curl    |  ❌   |  ❌   |   ❌    |   ❌    |
| Apache ab |  ❌   |  ❌   |   ❌    |   ❌    |
|   JSON    |  ✅   |  ✅   |   ❌    |   ✅    |
|   YAML    |  ✅   |  ✅   |   ❌    |   ✅    |
|  GoTest   |  ❌   |  ❌   |   ❌    |   ❌    |
|  PyTest   |  ❌   |  ❌   |   ❌    |   ❌    |

> 本文作者：卜卜星（HttpRunner 核心开发者）<br/>
> HttpRunner 项目官网: https://httprunner.com/<br/>
> 如果 HttpRunner 对你有过帮助，麻烦帮忙给个 ⭐️star⭐️ 鼓励下吧<br/>
> https://github.com/httprunner/httprunner<br/>
> 当前 HttpRunner 在参与 MTSC 2022 年度开源项目评选，HttpRunner 期待你的支持和鼓励！<br/>
> https://httprunner.com/blog/mtsc-2022-opensource/
