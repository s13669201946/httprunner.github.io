---
title: 变量（variables）
weight: 1
description: 基于变量机制实现参数的生命周期管理
---
在测试用例内部，HttpRunner 划分了两层变量空间作用域（context）。
- config：作为整个测试用例的全局配置项，作用域为整个测试用例；
- test：测试步骤的变量空间（context）会继承或覆盖 config 中定义的内容；
    - 若某变量在 config 中定义了，在某 test 中没有定义，则该 test 会继承该变量
    - 若某变量在 config 和某 test 中都定义了，则该 test 中使用自己定义的变量值
- 各个测试步骤（test）的变量空间相互独立，互不影响；
- 如需在多个测试步骤（test）中传递参数值，则需要使用 extract 关键字，并且只能从前往后传递


#### test
   在测试步骤（test)下声明的变量，作用域只在当前测试步骤（test)下有效。
声明变量用 variables，变量和对应值用键值对。例如：
```text
   teststeps:
   -
       name: step login
       variables:
           user: test
           psw: "123456"
```
在上述例子中 ，variables定义的局部变量，作用域为测试步骤中。

#### config
   在config下声明变量(variables)是全局变量，整个测试用例（testcase）的所有地方均可以引用，例如
```text
   "config": {
       "name": "testcase description",
       "parameters": [
           {"user_agent": ["iOS/10.1", "iOS/10.2", "iOS/10.3"]},
           {"app_version": "${P(app_version.csv)}"},
           {"os_platform": "${get_os_platform()}"}
       ],
       "variables": [
           {"user_agent": "iOS/10.3"},
           {"device_sn": "${gen_random_string(15)}"},
           {"os_platform": "ios"}
       ],
       "request": {
           "base_url": "http://127.0.0.1:5000",
           "headers": {
               "Content-Type": "application/json",
               "device_sn": "$device_sn"
           }
       },
       "output": [
           "token"
       ]
   }

```
在上述例子中 ，variables定义的全局变量，作用域为整个用例。
#### 变量优先级
原则上，尽可能使用不同的变量名称， 当变量之间出现重复名称时，就需要您了解优先级策略。例如：
```text
config:
    name: xxx
    variables:              # config variables
        varA: "configA"
        varB: "configB"
        varC: "configC"
    parameters:             # parameter variables
        varA: ["paramA1"]
        varB: ["paramB1"]

teststeps:
-
    name: step 1
    variables:              # step variables
        varA: "step1A"
    request:
        url: /$varA/$varB/$varC # varA="step1A", varB="paramB1", varC="configC"
        method: GET
    extract:                # extracted variables
        varA: body.data.A   # suppose varA="extractVarA"
        varB: body.data.B   # suppose varB="extractVarB"
-
    name: step 2
    varialbes:
        varA: "step2A"
    request:
        url: /$varA/$varB/$varC # varA="step2A", varB="extractVarB", varC="configC"
        method: GET

```
在以上用例中，变量优先级按以下顺序排列：

- 步骤变量 > 提取变量，例如步骤 2，varA="step2A"

- 参数变量 > 配置变量，例如第 1 步，varB="paramB1"

- 提取变量 > 参数变量 > 配置变量，例如第 2 步，varB="extractVarB"

- 配置变量的优先级最低，例如第 1/2 步，varC="configC"

#### 测试套件优先级
```text
config:
    name: xxx
    variables:                  # testsuite config variables
        varA: "configA"
        varB: "configB"
        varC: "configC"

testcases:
-
    name: case 1
    variables:                  # testcase variables
        varA: "case1A"
    testcase: /path/to/testcase1
    export: ["varA", "varB"]    # export variables
-
    name: case 2
    varialbes:                  # testcase variables
        varA: "case2A"
    testcase: /path/to/testcase2
```
在测试套件中，变量优先级按以下顺序排列：

- 测试用例变量 > 导出变量 > 测试套件配置变量 > 引用的测试用例配置变量