---
title: 脚手架创建项目
weight: 3
description: 介绍 HttpRunner 创建脚手架项目的几种模式
---

## 参数概览

```text
$ hrp startproject -h
create a scaffold project

Usage:
  hrp startproject $project_name [flags]

Flags:
      --go              generate hashicorp go plugin
  -h, --help            help for startproject
      --ignore-plugin   ignore function plugin
      --py              generate hashicorp python plugin (default true)

Global Flags:
      --log-json           set log to json format
  -l, --log-level string   set log level (default "INFO")
```

## 选择 go 插件

```bash
$ hrp startproject demo --go
```

## 选择 python 插件

```bash
$ hrp startproject demo --py
```

## 忽略插件

```bash
$ hrp startproject demo --ignore-plugin
```
