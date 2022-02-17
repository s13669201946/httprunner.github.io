---
title: "安装说明"
weight: 3
description: 本文介绍了 HttpRunner 的几种安装方式。
---

## 安装命令行工具

`HttpRunner+` 采用 Golang 开发，相比于之前的 Python 版本，最大的一个优势是可以编译生成二进制文件。在目标系统只需要下载到对应系统环境的二进制文件即可运行，无需安装任何运行时环境依赖（例如 Python、Java JDK、NodeJS 等）。

### 一键部署（推荐）

为了加速二进制包的下载速度，我们已经将编译产物上传到了阿里云 OSS，并且提供了一键安装部署的脚本。只需执行一条 shell 命令，即可完成 hrp 的下载和安装操作。

> 注：大多数 Linux/macOS 系统中都会预置 curl/wget 命令行工具，如果你的系统中未安装，需自行解决。

```bash
# install via curl
$ bash -c "$(curl -ksSL https://httprunner.oss-cn-beijing.aliyuncs.com/install.sh)"
# install via wget
$ bash -c "$(wget https://httprunner.oss-cn-beijing.aliyuncs.com/install.sh -O -)"
```

### 自行下载安装

同时，你也可以在 [GitHub Releases][releases] 页面中，自行选择版本进行下载。

当前 HttpRunner+ 在每次发布版本时，会自动编译生成 5 个版本，覆盖的环境包括：

- macOS(darwin) + amd64(x86)
- macOS(darwin) + arm64(M1)
- linux + amd64(x86)
- linux + arm64
- windows + amd64(x86)

如果你的系统环境不在这个范围内，那么你只能自行拉取源码进行编译了。

```bash
# 拉取 hrp 源码
$ git clone https://github.com/httprunner/hrp.git
$ cd hrp
# 通过 make 进行一键编译，生成的产物在 output 文件夹中
$ make build
[info] build hrp cli tool
++ mkdir -p output
++ bin_path=output/hrp
++ go build -ldflags '-s -w' -o output/hrp cli/hrp/main.go
++ ls -lh output/hrp
-rwxr-xr-x  1 debugtalk  staff    16M Feb 17 11:52 output/hrp
++ chmod +x output/hrp
++ ./output/hrp -v
hrp version v0.6.0
```

获取到编译产物后，你只需给 `hrp` 添加可运行权限即可。同时推荐将 `hrp` 移动到系统 bin 目录，方便全局调用。

```bash
$ chmod +x hrp
$ mv hrp /user/local/bin
```

### 检查安装结果

完成安装操作后，你将获得一个 `hrp` 命令行工具。`hrp` 包含多个子命令，具体的使用方式可查看命令行帮助。

在你的命令行终端中执行 `hrp -h` 命令，如果能正常打印帮助信息，则说明 `hrp` 已安装成功。

```text
$ hrp -h
hrp (HttpRunner+) is the next generation for HttpRunner. Enjoy! ✨ 🚀 ✨

License: Apache-2.0
Github: https://github.com/httprunner/hrp
Copyright 2021 debugtalk

Usage:
  hrp [command]

Available Commands:
  boom        run load test with boomer
  completion  generate the autocompletion script for the specified shell
  har2case    Convert HAR to json/yaml testcase files
  help        Help about any command
  run         run API test

Flags:
  -h, --help               help for hrp
      --log-json           set log to json format
  -l, --log-level string   set log level (default "INFO")
  -v, --version            version for hrp

Use "hrp [command] --help" for more information about a command.
```

## 安装依赖包（开发者模式）

`HttpRunner+` 除了可以作为命令行工具提供给用户进行使用，还可以作为库函数，供开发者调用进行二次开发。

当前 `HttpRunner+` 支持 Golang `1.16+` 和主流操作系统（Linux/macOS/Windows），我们通过在 [GitHub-Actions][github-actions] 配置 CI 进行了兼容性测试保障。

通过如下命令可安装依赖包：

```bash
$ go get -u github.com/httprunner/hrp
```

然后你就可以在你的工程中导入 `github.com/httprunner/hrp` 进行 Golang 用例编写或者二次开发了。


[releases]: https://github.com/httprunner/hrp/releases
[github-actions]: https://github.com/httprunner/hrp/actions
