---
title: hrp run
weight: 1
description: 通过 hrp run 命令行工具启动接口测试
---

```text
$ hrp run -h
run yaml/json testcase files for API test

Usage:
  hrp run $path... [flags]

Examples:
  $ hrp run demo.json   # run specified json testcase file
  $ hrp run demo.yaml   # run specified yaml testcase file
  $ hrp run examples/   # run testcases in specified folder

Flags:
  -c, --continue-on-failure   continue running next step when failure occurs
  -g, --gen-html-report       generate html report
  -h, --help                  help for run
      --http-stat             turn on HTTP latency stat (DNSLookup, TCP Connection, etc.)
      --log-plugin            turn on plugin logging
      --log-requests-off      turn off request & response details logging
  -p, --proxy-url string      set proxy url
  -s, --save-tests            save tests summary

Global Flags:
      --log-json           set log to json format
  -l, --log-level string   set log level (default "INFO")
```
