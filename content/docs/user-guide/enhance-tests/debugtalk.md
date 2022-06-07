---
title: 自定义函数（debugtalk.xx）
weight: 3
description: 基于自定义函数可辅助实现任意复杂的业务逻辑测试
---

HttpRunner v4 开始支持全新的插件化机制（plugin），得益于 gRPC 服务支持，我们几乎可以用任何语言编写插件代码，这部分能力已经单独抽离出了一个基础组件，详见 [httprunner/funplugin]。

目前，我们已经支持 python、go 语言两种自定义函数编写方式。其中 go 代码编写，要求编译为 debugtalk.bin 文件。插件文件 debugtalk.xx 要求必须放在项目根目录中。

接下来，介绍如何通过 go 与 python 两种自定义函数编写方式实现任意复杂的业务逻辑测试。

## plugin-go

使用 go 语言编写方式，一共需要进行两步操作：

1. 编写 debugtalk.go；
2. 编译 debugtalk.go 生成 debugtalk.bin。

### 编写 dubugtalk.go

编写 debugtalk.go 需要注意如下两点：

1. 包名必须为 package main
2. 函数名推荐使用驼峰命名、首写字母需要大写

```go
package main

import (
	"fmt"
)

func SumTwoInt(a, b int) int {
	return a + b
}

func SumInts(args ...int) int {
	var sum int
	for _, arg := range args {
		sum += arg
	}
	return sum
}

func Sum(args ...interface{}) (interface{}, error) {
	var sum float64
	for _, arg := range args {
		switch v := arg.(type) {
		case int:
			sum += float64(v)
		case float64:
			sum += v
		default:
			return nil, fmt.Errorf("unexpected type: %T", arg)
		}
	}
	return sum, nil
}

func SetupHookExample(args string) string {
	return fmt.Sprintf("step name: %v, setup...", args)
}

func TeardownHookExample(args string) string {
	return fmt.Sprintf("step name: %v, teardown...", args)
}

func GetUserAgent() string {
	return "hrp/fungo"
}

```

### 生成 debugtalk.bin

编写完 debugtalk.go 后，还需要手动编译成 hrp 可直接调用的插件产物。我们可以使用如下命令进行编译：

```bash
$ hrp build debugtalk.go -o dest_path  // dest_path 为项目根路径
```

使用 hrp build 编译会在 debugtalk.go 所在目录下生成 debugtalk_gen.go 与 go.mod 等编译中间产物，用户无需关心。编译完成后会在指定的项目根目录下生成 hrp 可直接调用的 debugtalk.bin 文件。

到此，我们就可以在测试用例中调用刚刚编写好的自定义函数了。调用的函数名原则上应该与 debugtalk.go 文件中定义的函数名保持一致，但为了兼顾命名的通用性，hrp 对调用函数时使用驼峰与蛇形命名同时做了兼容。

## plugin-python

使用 python 语言编写方式，仅需在项目根目录 debugtalk.py 中编写自定义函数即可，无需进行额外编译。同时，使用 hrp run/boom 也兼容 v4.0 版本之前的写法。

```python
import logging
import time
from typing import List


def get_user_agent():
    return "hrp/funppy"


def sleep(n_secs):
    time.sleep(n_secs)


def sum(*args):
    result = 0
    for arg in args:
        result += arg
    return result


def sum_ints(*args: List[int]) -> int:
    result = 0
    for arg in args:
        result += arg
    return result


def sum_two_int(a: int, b: int) -> int:
    return a + b


def sum_two_string(a: str, b: str) -> str:
    return a + b


def sum_strings(*args: List[str]) -> str:
    result = ""
    for arg in args:
        result += arg
    return result


def concatenate(*args: List[str]) -> str:
    result = ""
    for arg in args:
        result += str(arg)
    return result


def setup_hook_example(name):
    logging.warning("setup_hook_example")
    return f"setup_hook_example: {name}"


def teardown_hook_example(name):
    logging.warning("teardown_hook_example")
    return f"teardown_hook_example: {name}"

```

注意：使用 hrp 进行测试时，hrp 会根据 debugtalk.py 文件生成程序可直接调用的 .debugtalk_gen.py 文件，请勿编辑。

[httprunner/funplugin]: https://github.com/httprunner/funplugin