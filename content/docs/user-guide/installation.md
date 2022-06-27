---
title: 安装说明
weight: 1
description: HttpRunner 的几种安装方式
---

## 安装命令行工具

`HttpRunner v4` 采用 Golang 开发，相比于之前的 Python 版本，最大的一个优势是可以编译生成二进制文件。在目标系统只需要下载到对应系统环境的二进制文件即可运行，无需安装任何运行时环境依赖（例如 Python、Java JDK、NodeJS 等）。

当前 HttpRunner v4 支持如下几种安装方式。

### 一键部署（推荐）

为了加速二进制包的下载速度，我们已经将编译产物上传到了阿里云 OSS，并且提供了一键安装部署的脚本。只需执行一条 shell 命令，即可完成 hrp 的下载和安装操作。

```bash
$ bash -c "$(curl -ksSL https://httprunner.com/script/install.sh)"
```

注意：`install.sh` 脚本内部依赖 `curl/tar/mktemp/ls/rm/uname/chmod/command` 命令行工具，大多数 `Linux/macOS` 系统都会预置，如果你的系统中存在缺失，需自行解决。

针对 `Windows` 系统，上述脚本较大概率无法正常运行，建议自行下载编译产物后进行配置。

### 下载编译产物

同时，你也可以在 [GitHub Releases][releases] 页面中，自行选择版本进行下载。

当前 HttpRunner v4 在每次发布版本时，会自动编译生成 5 个版本，覆盖的环境包括：

- macOS(darwin) + amd64(x86)
- macOS(darwin) + arm64(M1)
- linux + amd64(x86)
- linux + arm64
- windows + amd64(x86)

获取到编译产物（`.tar.gz` 格式）后，对压缩包进行解压：

```bash
$ tar -xzf hrp-xxx.tar.gz
```

解压后可以获得一个 `hrp` 二进制文件，你只需给 `hrp` 添加可运行权限即可。

```bash
$ chmod +x hrp
```

同时为了让 `hrp` 在系统中可以全局调用，推荐将 `hrp` 添加到系统环境变量的 `PATH` 路径中，

针对 `Linux/macOS` 系统，推荐将 `hrp` 移动到系统 `/user/local/bin` 目录。

```bash
$ mv hrp /user/local/bin/
```

针对 `Windows` 系统：

- 在 C 盘根目录下创建 `HttpRunner` 目录（自定义目录），将 `hrp.exe` 文件放在该目录下
- 在「我的电脑=>属性=>高级系统设置=>环境变量」配置中，在 PATH 下新增系统变量，将 `C:\\HttpRunner` 写入 PATH

如果你的环境为 MinGW64，根据用户反馈，下载 `windows-amd64` 版本也是可以使用的。

### 自行本地编译

如果在上述已有的编译产物中没有包含你的系统类型，那么你可以自行拉取源码进行编译。

```bash
# 拉取 hrp 源码
$ git clone https://github.com/httprunner/httprunner.git
$ cd httprunner
# 通过 make 进行一键编译，生成的产物在 output 文件夹中
$ make build
[info] build hrp cli tool
++ mkdir -p output
++ bin_path=output/hrp
++ go build -ldflags '-s -w' -o output/hrp hrp/cmd/cli/main.go
++ ls -lh output/hrp
-rwxr-xr-x  1 debugtalk  staff    20M Apr 10 18:18 output/hrp
++ chmod +x output/hrp
++ ./output/hrp -v
hrp version v4.0.0-alpha
```

### go install 安装

如果你的系统有 Golang 环境，那么也可以通过 `go install` 命令从 GitHub 仓库中拉取代码进行安装。

指定版本号（v4.x.y）进行安装：

```bash
$ go install github.com/httprunner/httprunner/v4/hrp/cmd/cli@v4.x.y
```

如果你期望使用最新的代码进行安装，可以指定 master 分支进行安装：

```bash
$ go install github.com/httprunner/httprunner/v4/hrp/cmd/cli@master
```

### 检查安装结果

完成安装操作后，你将获得一个 `hrp` 命令行工具。`hrp` 包含多个子命令，具体的使用方式可查看命令行帮助。

在你的命令行终端中执行 `hrp -h` 命令，如果能正常打印帮助信息，则说明 `hrp` 已安装成功。

```text
$ hrp -h

██╗  ██╗████████╗████████╗██████╗ ██████╗ ██╗   ██╗███╗   ██╗███╗   ██╗███████╗██████╗
██║  ██║╚══██╔══╝╚══██╔══╝██╔══██╗██╔══██╗██║   ██║████╗  ██║████╗  ██║██╔════╝██╔══██╗
███████║   ██║      ██║   ██████╔╝██████╔╝██║   ██║██╔██╗ ██║██╔██╗ ██║█████╗  ██████╔╝
██╔══██║   ██║      ██║   ██╔═══╝ ██╔══██╗██║   ██║██║╚██╗██║██║╚██╗██║██╔══╝  ██╔══██╗
██║  ██║   ██║      ██║   ██║     ██║  ██║╚██████╔╝██║ ╚████║██║ ╚████║███████╗██║  ██║
╚═╝  ╚═╝   ╚═╝      ╚═╝   ╚═╝     ╚═╝  ╚═╝ ╚═════╝ ╚═╝  ╚═══╝╚═╝  ╚═══╝╚══════╝╚═╝  ╚═╝

HttpRunner is an open source API testing tool that supports HTTP(S)/HTTP2/WebSocket/RPC
network protocols, covering API testing, performance testing and digital experience
monitoring (DEM) test types. Enjoy! ✨ 🚀 ✨

License: Apache-2.0
Website: https://httprunner.com
Github: https://github.com/httprunner/httprunner
Copyright 2021 debugtalk

Usage:
  hrp [command]

Available Commands:
  boom         run load test with boomer
  completion   generate the autocompletion script for the specified shell
  har2case     convert HAR to json/yaml testcase files
  help         Help about any command
  pytest       run API test with pytest
  run          run API test with go engine
  startproject create a scaffold project

Flags:
  -h, --help               help for hrp
      --log-json           set log to json format
  -l, --log-level string   set log level (default "INFO")
  -v, --version            version for hrp

Use "hrp [command] --help" for more information about a command.
```

## 安装依赖包（开发者模式）

`HttpRunner` 除了可以作为命令行工具提供给用户进行使用，还可以作为库函数，供开发者调用进行二次开发。

当前 `HttpRunner` 支持 Golang `1.16+` 和主流操作系统（Linux/macOS/Windows），我们通过在 [GitHub-Actions][github-actions] 配置 CI 进行了兼容性测试保障。

通过如下命令可安装依赖包：

```bash
$ go get -u github.com/httprunner/httprunner/v4
```

然后你就可以在你的工程中导入 `github.com/httprunner/httprunner/v4/hrp` 进行 Golang 用例编写或者二次开发了。

```go
package tests

import (
	"testing"

	"github.com/httprunner/httprunner/v4/hrp"
)

func TestCaseCallFunction(t *testing.T) {
	testcase := &hrp.TestCase{
		Config: hrp.NewConfig("run request with functions").
			SetBaseURL("https://postman-echo.com").
			WithVariables(map[string]interface{}{
				"n": 5,
				"a": 12.3,
				"b": 3.45,
			}).
			SetVerifySSL(false),
		TestSteps: []hrp.IStep{
			hrp.NewStep("get with params").
				GET("/get").
				WithParams(map[string]interface{}{"foo1": "${gen_random_string($n)}", "foo2": "${max($a, $b)}", "foo3": "Foo3"}).
				WithHeaders(map[string]string{"User-Agent": "HttpRunnerPlus"}).
				Extract().
				WithJmesPath("body.args.foo1", "varFoo1").
				Validate().
				AssertEqual("status_code", 200, "check status code").
				AssertLengthEqual("body.args.foo1", 5, "check args foo1").
				AssertEqual("body.args.foo2", "12.3", "check args foo2").
				AssertTypeMatch("body.args.foo3", "str", "check args foo3 is type string").
				AssertStringEqual("body.args.foo3", "foo3", "check args foo3 case-insensitivity").
				AssertContains("body.args.foo3", "Foo", "check contains ").
				AssertContainedBy("body.args.foo3", "this is Foo3 test", "check contained by"), // notice: request params value will be converted to string
			hrp.NewStep("post json data with functions").
				POST("/post").
				WithHeaders(map[string]string{"User-Agent": "HttpRunnerPlus"}).
				WithBody(map[string]interface{}{"foo1": "${gen_random_string($n)}", "foo2": "${max($a, $b)}"}).
				Validate().
				AssertEqual("status_code", 200, "check status code").
				AssertLengthEqual("body.json.foo1", 5, "check args foo1").
				AssertEqual("body.json.foo2", 12.3, "check args foo2"),
		},
	}

	err := hrp.NewRunner(t).Run(testcase)
	if err != nil {
		t.Fatalf("run testcase error: %v", err)
	}
}
```

## 依赖环境说明
- go版本 : go 1.16以上
- Python版本 : Python3.7 / Python3.8 / Python3.9 / Python3.10

- 依赖库：
       | **依赖库**               | **版本要求**        | **是否必须** |
       |:---------------------:|:---------------:|:--------:|
       | **python**            | 3.7及其以上         | 是        |
       | **requests**          | 2.22.0及其以上      | 是        |
       | **pyyaml**            | 5.4.1及其以上       | 是        |
       | **pydantic**          | 大于1.8.0，小于1.9.0 | 是        |
       | **loguru**            | 0.4.1及其以上       | 是        |
       | **jmespath**          | 0.9.5及其以上       | 是        |
       | **black**             | 22.3.0及其以上      | 是        |
       | **pytest**            | 7.1.1及其以上       | 是        |
       | **pytest-html**       | 3.1.1及其以上       | 是        |
       | **sentry-sdk**        | 0.14.4及其以上      | 是        |
       | **allure-pytest**     | 2.8.16及其以上      | 否        |
       | **requests-toolbelt** | 0.9.1及其以上       | 否        |
       | **filetype**          | 1.0.7及其以上       | 否        |
       | **Brotli**            | 1.0.9及其以上       | 是        |
       | **jinja2**            | 3.0.3及其以上       | 是        |
       | **toml**              | 0.10.2及其以上      | 是        |
       | **sqlalchemy**        | 1.4.36及其以上      | 否        |


[releases]: https://github.com/httprunner/httprunner/releases
[github-actions]: https://github.com/httprunner/httprunner/actions
