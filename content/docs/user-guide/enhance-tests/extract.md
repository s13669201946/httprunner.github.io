---
title: 参数提取（extract）
weight: 2
description: 基于参数提取机制实现响应结果字段提取和参数关联
---
目前，HttpRunner支持多种响应结果字段提取方式：
- 若响应结果为 JSON 结构，可采用.运算符的方式，例如headers.Content-Type、content.success；
- 若响应结果为 text/html 结构，可采用正则表达式的方式，例如blog-motto\">(.*)；
- 当参数提取出来后，可采用 "$ 变量名 " 的方式进行参数引用。

例如：
```text
"teststeps": [
		{
			"name": "",
			"variables": {
				"name": "demo"
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
			"validate": [],
			"extract": {
				"name": "body.headers.Name"
			},
			"setup_hooks": [],
			"teardown_hooks": []
		}
	]

```
分析：上述例子中，extract是从当前 HTTP 请求的响应结果中提取参数Name，并保存到参数变量中（例如name），后续测试用例可通过 $name 的形式进行引用。