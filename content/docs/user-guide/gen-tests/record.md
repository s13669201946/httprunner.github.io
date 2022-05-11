---
title: 录制生成
weight: 1
description: 基于 HAR 生成测试用例
---

```text
$ hrp har2case -h
convert HAR to json/yaml testcase files

Usage:
  hrp har2case $har_path... [flags]

Flags:
  -h, --help                help for har2case
  -d, --output-dir string   specify output directory, default to the same dir with har file
  -p, --profile string      specify profile path to override headers and cookies
  -j, --to-json             convert to JSON format (default true)
  -y, --to-yaml             convert to YAML format

Global Flags:
      --log-json           set log to json format
  -l, --log-level string   set log level (default "INFO")
```
