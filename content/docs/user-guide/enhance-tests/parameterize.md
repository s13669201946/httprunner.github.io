---
title: 参数化数据驱动（parameterize） 
weight: 4 
description: 基于参数化数据驱动可实现多组数据驱动测试执行
---

> v4.0 版本开始，对标 LoadRunner，参数化数据驱动支持对每个参数设置参数选取策略，具体策略包括sequential、random与unique。

为了让大家更快速的理解，本节将从数据结构与使用示例出发，详细介绍如何配置参数化数据驱动。

参数化数据驱动的数据结构如下：

```go
type TConfig struct {
    ...
    Parameters        map[string]interface{} `json:"parameters,omitempty" yaml:"parameters,omitempty"`
    ParametersSetting *TParamsConfig         `json:"parameters_setting,omitempty" yaml:"parameters_setting,omitempty"`
    ...
}

type TParamsConfig struct {
    PickOrder  iteratorPickOrder           `json:"strategy,omitempty" yaml:"strategy,omitempty"`
    Strategies map[string]iteratorStrategy `json:"strategies,omitempty" yaml:"strategies,omitempty"`
    Limit      int                         `json:"limit,omitempty" yaml:"limit,omitempty"`
}

type iteratorPickOrder string

const (
    pickOrderSequential iteratorPickOrder = "sequential"
    pickOrderRandom     iteratorPickOrder = "random"
    pickOrderUnique     iteratorPickOrder = "unique"
)

type iteratorStrategy struct {
    Name      string            `json:"name,omitempty" yaml:"name,omitempty"`
    PickOrder iteratorPickOrder `json:"pick_order,omitempty" yaml:"pick_order,omitempty"`
}
```

参数化数据驱动使用示例：

```yml
config:
  ...
  parameters:
    user_agent: [ "iOS/10.1", "iOS/10.2" ]
    username-password: ${parameterize($file)}
    user_id: ${get_user_id(10)}
  parameters_setting:
    pick_order: "random"
    strategies:
      user_agent:
        name: "user-identity"
        pick_order: "sequential"
      username-password:
        name: "user-info"
        pick_order: "random"
    limit: 6
  variables:
    file: examples/hrp/account.csv
  ...

teststeps:
  ...
```

参数化数据驱动使用说明：

- parameters（支持 3 种输入方式）
  - 参数列表，形如：`user_agent: [ "iOS/10.1", "iOS/10.2" ]`
  - csv文件，形如：`username-password: ${parameterize($file)}`，csv 示例：[account.csv](https://github.com/httprunner/httprunner/blob/master/examples/hrp/account.csv)
  - 自定义函数，形如：`user_id: ${get_user_id(10)}`，函数返回值为列表
  
  说明：username-password参数名称采用短横线（-）作为分隔符，表示从 csv 中读取 username 与 password 这两个参数。如果直接采用列表方式导入参数，username-password需对应一个二维列表，自定义函数方式同理。

- parameters_setting
  - pick_order：整体策略，如果未单独指定参数选取策略，则默认使用整体策略
  - strategies：单独配置每个参数的策略
  - limit：迭代次数

针对「parameters_setting」，有 2 个需要特别注意的点：
- strategies 可以为每一个参数配置参数名称与具体选取策略。参数名称仅用于标识，可选填。如果未设置参数选取策略，则默认使用 sequential 策略。
- 迭代次数默认为所有顺序选取执行参数的笛卡尔积，我们也可以通过设置 limit 来指定迭代次数，有效的迭代次数为 limit > 0，如果 limit = 0 表示默认，如果 limit < 0，则表示无限制迭代次数。