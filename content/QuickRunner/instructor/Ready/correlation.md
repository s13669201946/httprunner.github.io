---
title: "关联机制"
weight: 5
type: docs
---
若您在脚本执行期间，想使各接口请求在上下文之间可以进行参数传递，那么您就需要使用到 QuickRunner 的「关联机制」这个功能。
目前，「关联机制」支持三种形式的来源检查：status_code(响应码)、body(响应体)、headers（响应头），同时支持两种形式的提取规则：正则匹配、json路径。

该功能的使用方式如下：
### 提取参数

####  json_path形式

<img src="/image/QuickRunner/direction/correlation_json_path.png" alt="QuickRunner" width="800">


#### 正则表达式

使用方式同 json_path 一致，当前仅支持对 body 内容进行正则匹配，可提取 HTML 内容。
<img src="/image/QuickRunner/direction/correlation_regx.png" alt="QuickRunner" width="800">

### 参数引用

参数引用需要使用  $参数名 的形式。
<img src="/image/QuickRunner/direction/correlation_apply.png" alt="QuickRunner" width="800">
