---
title: 变量（variables）
weight: 1
description: 基于变量机制实现参数的生命周期管理
---

在测试用例中，很多时候我们需要对参数进行声明和引用，这就需要用到变量（variables）机制。

## 变量声明

在 HttpRunner 测试用例中，有 4 个地方可以对变量进行声明。

### 声明全局变量（config variables）

在 `config` 下声明的 `variables` 为测试用例全局变量，作用域为整个测试用例，在测试用例的所有地方都可以引用。

### 声明数据驱动（parameters）

在 `config` 下声明的 `parameters` 为测试用例的驱动参数；它的作用域也是覆盖整个测试用例，在测试用例的所有地方都可以引用。

### 声明局部变量（teststeps variables）

在单个测试步骤（`teststep`）下声明的 `variables` 是测试步骤局部变量，作用域仅限当前步骤。

各个测试步骤的变量相互独立，互不影响。

### 提取参数变量（session variables）

还有一种变量声明的方式，可以在某个测试步骤（teststep）中提取（extract）特定的响应参数，并赋值给指定的变量名。该操作也常被成为 `参数关联`。

提取的参数变量类似于 session 参数，作用域为当前步骤及之后的步骤。

## 变量引用

在 HttpRunner 的测试用例中，约定通过 `${}` 或 `$` 的形式来引用变量。

例如：`$var` 或 `${var}`

大多数情况下，采用 `$var` 或 `${var}` 这两种形式都是可以的。但如果在某些字段中存在部分引用变量的情况，例如 `abc123def` 中 `123` 需要引用变量，那么就只能使用 `${var}` 的形式，即 `abc${num}def`；如果使用 `abc$numdef` 的话，变量名称会被识别为 `numdef`。

另一种需要说明的情况，如果在测试用例中本身就存在 `$` 符号，那么可以通过 `$$` 进行转义。

例如，测试用例中某个字段的原始内容为 `$m`，那么为了避免将其解析为变量，则需要将其写为 `$$m`。

## 变量优先级

针对上述的 4 类变量类型，如果声明的变量名称出现重复，则会按照一定的优先级策略进行处理。

优先级从高到低依次为：step variables > session variables > parameter variables > config variables

## 示例

下面通过一个完整的示例进行说明。

```text
config:{
    "name": xxx
    "variables":  {			# config variables
        "varA": "configA"
        "varB": "configB"
        "varC": "configC"
    }
    "parameters" : {		# parameters variables
		"varA-varB": [
			["paramA1", "paramB1"],
			["paramA2", "paramB2"],
		]
    }
}

teststeps:[
	{
		"name": "step 1"
		"variables":  {		# step variables
			"varA": "step1A"
		},
		"request": {
			"method" : "GET"
			"url" : /$varA/$varB/$varC  # varA="step1A", varB="paramB1", varC="configC"
		}
		"extract": {
			"varA": "body.data.A", # suppose varA="extractVarA"
			"varB": "body.data.B"  # suppose varB="extractVarB"
		}
	},
	{
		"name": "step 2"
		"varialbes": {		# step variables
			"varA" : "step2A"
		}
		"request": {
			"url" : /$varA/$varB/$varC # varA="step2A", varB="extractVarB", varC="configC"
			"method" : "GET"
		}
	}
]
```

在步骤 1 中：

- varA 定义了局部变量，优先级最高，因此实际值为 step1A
- varB 未定义局部变量，将继承全局变量；而在全局变量中，parameter 的优先级高于 variables，因此 varB 实际值为 paramB1
- varC 未定义局部变量，将继承全局变量；在全局变量中，varC 仅在 variables 中进行了声明，因此 varC 的实际值为 configC

在步骤 2 中：

- varA 定义了局部变量，优先级最高，因此实际值为 step2A
- varB 未定义局部变量；而步骤 1 中有提取该变量，优先级高于全局变量，因此 varB 实际值为 extractVarB
- varC 未定义局部变量，也不存在同名 session 变量，将继承全局变量，因此 varC 实际值为 configC
