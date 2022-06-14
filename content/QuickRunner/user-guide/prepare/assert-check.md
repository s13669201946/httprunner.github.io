---
title: "断言&检查点"
weight: 9
type: docs
---
通过设置断言/检查点，可以检查请求返回结果是否符合用户的预期。这对于检测非标准错误很有用。

QuickRunner支持三种形式的来源检查 "status_code"、"body"、"headers"。<br/>

支持两种形式的提取规则：正则表达式，json 路径。

比较类型有：

|        比较类型 | 说明    |
|   ------------ | --------- |
|   equal        | 等于    |
|   greater_than | 大于    |
|   less_than    | 小于    |
|   not_equal    | 不等于 |
|   contains     | 包含    |
|   regex_match  | 正则匹配 |
|   startswith   | 以XXX开头 |
|   endswith     | 以XXX结尾 |
|   length_equal | 长度等于 |

该功能的使用方式如下：

<img src="/image/QuickRunner/direction/assert-check1.png" alt="QuickRunner" width="600">

<img src="/image/QuickRunner/direction/assert-check2.png" alt="QuickRunner" width="600">

#### 正则表达式

使用方式同 json_path 一致，当前仅支持对 body 内容进行正则匹配，可提取 HTML 内容。
<img src="/image/QuickRunner/direction/correlation_regx1.png" alt="QuickRunner" width="800">
<img src="/image/QuickRunner/direction/correlation_regx.png" alt="QuickRunner" width="800">
