---
title: 命令行工具
weight: 2
description: HttpRunner 的命令行工具及参数说明
---

HttpRunner v4.0 安装成功后，你将获得一个 `hrp` 命令行工具，执行 `hrp -h` 即可查看到参数帮助说明。

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
Copyright 2017 debugtalk

Usage:
  hrp [command]

Available Commands:
  boom         run load test with boomer
  completion   generate the autocompletion script for the specified shell
  convert      convert JSON/YAML testcases to pytest/gotest scripts
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

[详细命令行及参数说明][cmd]

[cmd]: https://github.com/httprunner/httprunner/blob/master/docs/cmd/hrp.md
