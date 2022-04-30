---
title: 版本对比
weight: 14
description: HttpRunner v4 与历史版本 v1/v2/v3/hrp 的对比
---

为了让你感观更加清晰，我整理出了如下表格，详细对比了各个版本间的关键差异点。

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

[jsonschema]: https://github.com/python-jsonschema/jsonschema
