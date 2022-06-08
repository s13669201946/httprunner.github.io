---
title: 转换生成
weight: 2
description: 使用主流 API 工具转换生成测试用例
---

## 将输入转化为 JSON/YAML/pytest/gotest

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