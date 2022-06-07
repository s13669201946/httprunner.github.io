---
title: 参数提取（extract）
weight: 2
description: 基于参数提取机制实现响应结果字段提取和参数关联
---

在实际业务场景中，很多时候存在参数关联的情况，即当前接口请求参数来自于之前接口的响应结果。

例如，通过手机号登录的场景中，登录接口请求参数需要带上服务端预先返回的短信验证码；如果缺少这个参数关联操作，接口调用将会失败。

目前，HttpRunner 支持 2 种响应结果字段提取方式。

提取的参数变量类似于 session 参数，作用域为当前步骤及之后的步骤，引用方式与普通的变量一致。

## jmespath 表达式

若响应结果为 JSON 结构，支持采用 [jmespath] 表达式进行参数提取。

[jmespath] 是一种 JSON 查询语言，可以使用非常灵活且强大的表达式查询 JSON 数据结构中的字段，并返回符合条件的数据。

针对 [jmespath] 官网中的示例：

```json
{
  "locations": [
    {"name": "Seattle", "state": "WA"},
    {"name": "New York", "state": "NY"},
    {"name": "Bellevue", "state": "WA"},
    {"name": "Olympia", "state": "WA"}
  ]
}
```

- 查询所有城市名称：`locations[*].name`
- 查询第二个城市名称：`locations[1].name` => "New York"
- 查询最后一个州名：`locations[-1].state` => "WA"

HttpRunner 完全继承了 [jmespath] 的强大能力，只需要重点注意两点：

1、extract 的对象仅有 5 种类型：

- status_code：提取响应状态码，例如 200、404
- proto：提取协议类型，例如 "HTTP/2.0"、"HTTP/1.1"
- headers：从响应 headers 中提取字段，例如 headers.name
- cookies：从响应 cookies 中提取字段，例如 cookies.Token
- body：从响应 body 中提取字段，例如 body.args.foo1

2、如果表达式中存在 `-` 的情况，那么需要加引号处理。

例如 `headers."Content-Type"`

```json
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
		}
	}
]
```

## 正则表达式（regex）

若响应结果为 text/html 格式，支持采用正则表达式的方式提取目标参数。

例如响应的 body 为：

```html
<html>
<title>参数提取（extract） | HttpRunner</title>
</html>
```

如果我们想提取页面的 title 字段，就可以这样做：`<title>(.*)</title>`

即在提取表达式中指定目标参数的左右边界，然后将目标参数替换为 `(.*)`；这样我们就能将正则匹配到的值赋值给参数变量了。

```json
"teststeps": [
	{
		"name": "",
		"variables": {
			"name": "demo"
		},
		"request": {
			"method": "GET",
			"url": "https://www.httpbin.org",
			"params": {},
			"headers": {
				"name": "$name",
				"Content-Type": "text/plain"
			}
		},
		"validate": [],
		"extract": {
			"title": "<title>(.*)</title>"
		}
	}
]
```


[jmespath]: https://jmespath.org/
