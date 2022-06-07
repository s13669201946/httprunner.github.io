---
title: 变量（variables）
weight: 1
description: 基于变量机制实现参数的生命周期管理
---
在测试用例内部，HttpRunner 划分了两层变量空间作用域（context）。
- config ：作为整个测试用例的全局配置项，作用域为整个测试用例；
- teststeps ：测试步骤的变量空间（context）会继承或覆盖 config 中定义的内容；
    - 若某变量在 config 中定义了，在某 teststeps 中没有定义，则该 teststeps 会继承该变量
    - 若某变量在 config 和某 teststeps 中都定义了，则该 teststeps 中使用自己定义的变量值
- 各个测试步骤（teststeps）的变量空间相互独立，互不影响；
- 如需在多个测试步骤（teststeps）中传递参数值，则需要使用 extract 关键字，并且只能从前往后传递


#### teststeps
   在测试步骤（teststeps)下声明的变量，作用域只在当前测试步骤（teststeps)下有效。
声明变量用 variables，变量和对应值用键值对。例如：
```text
   "teststeps": [
   		{
   			"name": "",
   			"variables": {
   				"name": "demo",
   				"sum": "${SumTwoInt(4,5)}"
   			},
   			"request": {
   				"method": "POST",
   				"url": "https://www.httpbin.org",
   				"params": {},
   				"headers": {
   					"name": "$name",
   					"Content-Type": "text/plain"
   				},
   				"body": ""
   			},
   		}
   	],
```
在上述例子中 ，variables 定义的局部变量，作用域为测试步骤中。

#### config
   在 config 下声明的 变量都是全局变量，全局变量分为参数变量 parameters ，配置变量 variables ，这些变量在整个测试用例（testcase）的所有地方均可以引用，例如
```text
   "config": {
   		"name": "request methods testcase: empty testcase",
   		"variables": {
   			"name": "demo"
   		},
   		"parameters": {
   			"user_name-password": [
   				[
   					"user1",
   					"123"
   				],
   				[
   					"user2",
   					"456"
   				]
   			]
   		},
   		"parameters_setting": {
   			"strategies": {
   				"user_name-password": {
   					"name": "var",
   					"pickOrder": "sequential"
   				}
   			}
   		}
   	},

```
#### 变量优先级
原则上，尽可能使用不同的变量名称， 当变量之间出现重复名称时，就需要您了解优先级策略。例如：
```text
config:{
    "name": xxx
    "variables":  {
        "varA": "configA"  # config variables
        "varB": "configB"
        "varC": "configC"
    }
    "parameters" : {
            "varA-varB": [
               	    [
               			"paramA1",
               			"paramB1"
               	    ]
            ]
    }
    "parameters_setting": {
       			"strategies": {
       				"varA-varB": {
       					"name": "var",
       					"pickOrder": "sequential"
       				}
       			}
       		}
}

teststeps:[
        {
            "name": "step 1"
            "variables":  {
                "varA": "step1A"   # step variables
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
            "varialbes": {
                "varA" : "step2A"
            }
            "request": {
                "url" : /$varA/$varB/$varC # varA="step2A", varB="extractVarB", varC="configC"
                "method" : "GET"
            }
        }
]
```
在以上用例中，变量优先级按以下顺序排列：

- 步骤变量 > 提取变量，例如步骤 2，varA="step2A"

- 参数变量 > 配置变量，例如第 1 步，varB="paramB1"

- 提取变量 > 参数变量 > 配置变量，例如第 2 步，varB="extractVarB"

- 配置变量的优先级最低，例如第 1/2 步，varC="configC"

- 总结：步骤变量 > 提取变量 > 参数变量 > 配置变量
