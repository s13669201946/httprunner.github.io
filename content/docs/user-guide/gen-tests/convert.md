---
title: 转换生成
weight: 2
description: 使用主流 API 工具转换生成测试用例
---

## 将输入转化为 JSON/YAML/pytest/gotest

```text
$ hrp convert -h
convert to JSON/YAML/gotest/pytest testcases

Usage:
  hrp convert $path... [flags]

Flags:
  -h, --help                help for convert
  -d, --output-dir string   specify output directory, default to the same dir with har file
  -p, --profile string      specify profile path to override headers (except for auto-generated headers) and cookies
      --to-gotest           convert to gotest scripts (TODO)
      --to-json             convert to JSON scripts (default true)
      --to-pytest           convert to pytest scripts
      --to-yaml             convert to YAML scripts

Global Flags:
      --log-json           set log to json format
  -l, --log-level string   set log level (default "INFO")
```
