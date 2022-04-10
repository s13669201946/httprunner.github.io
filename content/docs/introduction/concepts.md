---
title: "核心概念"
weight: 12
description: HttpRunner 仅有的几个核心概念
---

## 测试用例（TestCase）

从 2.0 版本开始，HttpRunner 开始对测试用例的定义进行进一步的明确，参考 [wiki][wiki_testcase] 上的描述。

> A test case is a specification of the inputs, execution conditions, testing procedure, and expected results that define a single test to be executed to achieve a particular software testing objective, such as to exercise a particular program path or to verify compliance with a specific requirement.

概括下来，一条测试用例（testcase）应该是为了测试某个特定的功能逻辑而精心设计的，并且至少包含如下几点：

- 明确的测试目的（achieve a particular software testing objective）
- 明确的输入（inputs）
- 明确的运行环境（execution conditions）
- 明确的测试步骤描述（testing procedure）
- 明确的预期结果（expected results）

对应地，HttpRunner 的测试用例描述方式进行如下设计：

- 测试用例应该是完整且独立的，每条测试用例应该是都可以独立运行的
- 在 HttpRunner 中，每个 `YAML/JSON/pytest/go test` 文件对应一条测试用例
- HttpRunner 以 `TestCase` 为核心，将任意测试场景抽象为`有序`步骤的集合

数据结构示例如下：

```go
type TestCase struct {
	Config    *TConfig
	TestSteps []IStep
}
```

包含且仅有两部分：

- Config：测试用例的公共配置部分，包括用例名称、base_url、参数化数据源、是否开启 SSL 校验等
- TestSteps：有序步骤的集合；采用了 `go interface` 的设计理念，支持进行任意协议和测试类型的拓展；步骤内容统一在 `Run` 方法中进行实现。

```go
type IStep interface {
	Name() string
	Type() StepType
	Struct() *TStep
	Run(*SessionRunner) (*StepResult, error)
}
```

> 这里仅以 go 代码作为示意，实际使用 HttpRunner 时对 golang 基础没有要求。<br/>
> 详细数据结构可参考：

## 插件化机制（plugin）

HttpRunner 用例的核心为 JSON 结构体，但很多测试场景中需要动态运算逻辑，例如计算签名 MD5、生成随机数据等。

为了解决该问题，HttpRunner 引入了动态函数运算的能力，即在执行测试用例的过程中，会从文本中解析出变量和函数，调用执行后再将结果拼接回文本字符串。

这里的动态函数运算就是通过插件化机制（plugin）来实现的。

在 v4 版本中，HttpRunner 的 golang 执行引擎支持如下 4 种插件方案：

- Golang plugin over gRPC：Go 代码编写，要求编译为 `debugtalk.bin` 文件
- Golang plugin over net/rpc：同上
- Python plugin over gRPC：Python 代码编写，无需编译，要求命名为 `debugtalk.py` 文件
- Golang official plugin：Go 代码编写，要求编译为 `debugtalk.so` 文件

这部分能力已经单独抽离出了一个基础组件，详见 [httprunner/funplugin]。

针对 v4 版本的 pytest 执行引擎，以及 v4 之前的 Python 版本，支持解析调用 Python 函数的形式：

- Python 代码编写，无需编译，要求命名为 `debugtalk.py` 文件

需要特别强调的是，上述插件文件 `debugtalk.xx` 要求必须放在项目根目录中。

## 项目根目录

在 HttpRunner 的测试用例中，有时候需要引用其它测试用例或者数据文件（csv）。考虑到脚本的简洁些和可移植性，引用路径需要使用相对路径。

基于约定大于配置的思路，HttpRunner 通过定位插件文件的位置来确定项目的根目录，以此作为相对路径的基准。同时，定位插件文件存在优先级，如下文件的优先级从上到下依次降低。

- debugtalk.bin
- debugtalk.py
- debugtalk.so

## 函数（functions）

在 HttpRunner 的测试用例中，约定通过 `${}` 的形式来调用插件函数。

```
"${func{$a, $b}}"
```

## 变量（variables）

在 HttpRunner 的测试用例中，约定通过 `${}` 或 `$` 的形式来引用变量。

`$abc` or `${abc}`

如果在测试用例中本身就存在 `$` 符号，那么可以通过 `$$` 进行转义。

例如，测试用例中某个字段的原始内容为 `$m`，那么为了避免将其解析为变量，则需要将其写为 `$$m`。

## 变量优先级

在 HttpRunner 测试用例中，变量类型有 3 种：

- config variables：在 config 中定义的变量
- step variables：在 test step 中定义的变量
- session variables：在测试用例运行时，从前置 step 提取（extract）的变量

这 3 种变量的优先级从高到低依次为：step variables > session variables > config variables


[wiki_testcase]: https://en.wikipedia.org/wiki/Test_case
[httprunner/funplugin]: https://github.com/httprunner/funplugin
