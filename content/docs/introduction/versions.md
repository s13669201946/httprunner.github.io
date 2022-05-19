---
title: 版本对比
weight: 4
description: HttpRunner 各版本的功能特性详细对比
---

HttpRunner 经过近 5 年的迭代，已经进入到 v4.0 版本了。

## v4 与历史版本的对比

通过如下表格，可详细了解各个版本间的关键差异点。

| 版本 | v1 | v2 | v3 | HttpRunner+ | v4 |
| :--: | :--: | :--: | :--: | :--: | :--: |
| 发布时间 | 2018.03.07 | 2019.01.01 | 2020.03.10 | 2021.11.18 | 2022.05.01 |
| 开发语言 | Python | Python | Python | Golang | Golang + Python |
| 版本号规范（semver） | ❌ | ✅ | ✅ | ✅ | ✅ |
| 网络协议 | HTTP(S)/1.1 | HTTP(S)/1.1 | HTTP(S)/1.1 | HTTP(S)/1.1 | **多协议** HTTP(S)/HTTP2/WebSocket/_TCP/RPC_ |
| 脚本转换工具 | HAR | HAR | HAR |HAR | HAR/_Postman/Swagger/Curl_ |
| 工程脚⼿架 | ❌ | ✅ | ✅ | ✅ | ✅ |
| 测试⽤例（集）格式 | v1 | v2 | v2 | v2 | v2 |
| 测试⽤例分层机制 | v1 | v2 | v2 | v2 | v2 |
| 脚本格式类型 | YAML/JSON | YAML/JSON | YAML/JSON/**pytest** | YAML/JSON | YAML/JSON/**pytest**/**gotest** |
| 脚本格式校验 | ❌ | [jsonschema] | ❌ | ❌ | _TODO_ |
| 脚本编写语法提示 | ❌ | ❌ | pytest 链式调用 | gotest 链式调用 | gotest 链式调用 + pytest 链式调用 |
| 脚本执行引擎 | Python unittest | Python unittest | Python **pytest** | Go 自研 | Go 自研 + Python **pytest** |
| 插件化语言（debugtalk.xx） | Python | Python | Python | **多语言**（Go/Python） | **多语言**（Go/Python/_Java/etc._） |
| 参数提取机制 | regex + 点分隔符 | jmespath + regex + 点分隔符 | jmespath | jmespath + regex | jmespath + regex |
| skip 机制 | ✅ | ❌ | ❌ | ❌ | _TODO_ |
| 接口测试报告 | html 自研（jinja2） | html 自研（jinja2） | pytest-html/allure | html 自研（Go template） | html 自研（Go template） + _pytest-html/allure_ |
| 性能测试引擎 | Python Locust | Python Locust | Python Locust | Go Boomer | Go Boomer |
| 运行环境依赖 | Python 2.7/3.3+ | Python 2.7/3.5+ | Python 3.7+ pytest | 无需依赖 | Go 引擎无需依赖<br/>pytest 引擎依赖 Python 3.7+ |
| 网络性能采集 | ❌ | ❌ | ❌ | ❌ | ✅ |
| 安装部署方式 | pip | pip | pip | curl/wget | curl/wget |

注：v4 中 _斜体_ 代表当前还未支持，但计划会实现。

从上面的表格可以看出，HttpRunner v4 颇有点**金刚葫芦娃**的意思，囊括了之前所有版本的功能，并且增加了很多新特性。

```
HttpRunner v4 = v2 + v3 + hrp + ...
```

在使用体验和用例格式兼容性方面，v4 也会与之前的 v2/v3/hrp 做到兼容，因此后续 HttpRunner 的维护工作都将转到 v4 版本，之前的版本将不再单独维护。

## v4 版本的 Go & Python 功能对比

HttpRunner v4.0 同时采用了 Golang/Python 两种编程语言，底层会有两套相对独立的执行引擎，目标是兼具 Golang 的高性能和 pytest 的丰富生态。

关键差异点对比如下：

| 引擎 | Go | Python |
| :--: | :--: | :--: |
| 脚本类型 | YAML/JSON/gotest | YAML/JSON/pytest |
| 网络协议 | **多协议** HTTP(S)/HTTP2/WebSocket/_TCP/RPC_ | HTTP(S) |
| 脚手架工具 | hrp startproject | / |
| 用例生成工具 | hrp har2case | / |
| 脚本转换工具 | hrp convert | / |
| 插件化语言 | **多语言**（Go/Python/_Java/etc._） | Python |
| 运行环境依赖 | 与插件语言相关，详见[依赖环境说明] | Python 3.7+ |
| 脚本编写语法提示 | gotest 链式调用 | pytest 链式调用 |
| 运行接口测试 | hrp run | hrp pytest |
| 运行性能测试 | hrp boom | / |
| 网络性能采集 | hrp run --http-stat | / |
| 接口测试报告 | html 自研（Go template） | pytest-html/allure |


[jsonschema]: https://github.com/python-jsonschema/jsonschema
[依赖环境说明]: /docs/user-guide/installation/#%E4%BE%9D%E8%B5%96%E7%8E%AF%E5%A2%83%E8%AF%B4%E6%98%8E
