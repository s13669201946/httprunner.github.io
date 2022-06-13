---
title: 结果校验（validate）
weight: 6
description: 针对 API 响应结果进行预期结果校验
---

## 概念

结果校验（又称断言）是测试用例中的重要组成部分，可以对测试用例在运行过程中是否得到了预期结果进行校验，例如对响应状态码进行断言，以及对响应 JSON 中的具体字段进行断言。

目前 HttpRunner 支持两种匹配目标参数的方式，并且内置了丰富的结果检验函数，可以实现满足众多的测试场景需求。

HttpRunner v4.0 结果校验部分的数据结构如下：

```go
type Validator struct {
	Check   string      `json:"check" yaml:"check"`
	Assert  string      `json:"assert" yaml:"assert"`
	Expect  interface{} `json:"expect" yaml:"expect"`
	Message string      `json:"msg,omitempty" yaml:"msg,omitempty"`
}
```

从上面的数据结构可以看出，结果校验内容包含了 4 个字段，分别是字段提取表达式、断言函数、预期结果以及提示信息：
- 字段提取表达式：用于提取目标字段用作断言函数的输入，支持 jmespath 表达式和正则表达式（regex）两种提取方式，填写方式请参考[参数提取（extract）]章节
- 断言函数：顾名思义，就是用于对目标字段与预期结果是否满足相等、大小、包含、被包含等关系进行断言的函数，目前支持内置函数，不支持自定义拓展
- 预期结果：指定断言的预期结果，用作断言函数的另一个输入
- 提示信息：当前断言操作对应的提示信息，该字段为选填

## 示例

下面以 YAML 测试用例为例，展示结果校验功能的写法示例：

```yaml
config:
  name: validation demo
teststeps:
    - name: get httpbin
      request:
        method: GET
        url: https://www.httpbin.org
      validate:
        - check: status_code            # target field, support both jmespath and regex
          assert: eq                    # assertion method, you can use builtin method or custom defined function
          expect: 200                   # expected value, supposed to match target field
          message: check status code    # note message, optional
```


## 内置结果校验函数

`assert` 部分支持填写的内置结果校验函数包括如下内容：

| `assert`                                         | Description                   | A(check), B(expect)     | examples                                     |
|--------------------------------------------------|-------------------------------|-------------------------|----------------------------------------------|
| `eq`, `equals`, `equal`                          | value is equal                | A == B                  | 9 eq 9                                       |
| `lt`, `less_than`                                | less than                     | A < B                   | 7 lt 8                                       |
| `le`, `less_or_equals`                           | less than or equals           | A <= B                  | 7 le 8, 8 le 8                               |
| `gt`, `greater_than`                             | greater than                  | A > B                   | 8 gt 7                                       |
| `ge`, `greater_or_equals`                        | greater than or equals        | A >= B                  | 8 ge 7, 8 ge 8                               |
| `ne`, `not_equal`                                | not equals                    | A != B                  | 6 ne 9                                       |
| `str_eq`, `string_equals`                        | string equals                 | str(A) == str(B)        | 123 str_eq '123'                             |
| `len_eq`, `length_equals`, `length_equal`        | length equals                 | len(A) == B             | 'abc' len_eq 3, [1,2] len_eq 2               |
| `len_gt`, `count_gt`, `length_greater_than`      | length greater than           | len(A) > B              | 'abc' len_gt 2, [1,2,3] len_gt 2             |
| `len_ge`, `count_ge`, `length_greater_or_equals` | length greater than or equals | len(A) >= B             | 'abc' len_ge 3, [1,2,3] len_gt 3             |
| `len_lt`, `count_lt`, `length_less_than`         | length less than              | len(A) < B              | 'abc' len_lt 4, [1,2,3] len_lt 4             |
| `len_le`, `count_le`, `length_less_or_equals`    | length less than or equals    | len(A) <= B             | 'abc' len_le 3, [1,2,3] len_le 3             |
| `contains`                                       | contains                      | [1, 2] contains 1       | 'abc' contains 'a', [1,2,3] len_lt 4         |
| `contained_by`                                   | contained by                  | A in B                  | 'a' contained_by 'abc', 1 contained_by [1,2] |
| `type_match`                                     | A and B are in the same type  | type(A) == type(B)      | 123 type_match 1                             |
| `regex_match`                                    | regex matches                 | re.match(B, A)          | 'abcdef' regex_match 'a\w+d'                 |
| `startswith`                                     | starts with                   | A.startswith(B) is True | 'abc' startswith 'ab'                        |
| `endswith`                                       | ends with                     | A.endswith(B) is True   | 'abc' endswith 'bc'                          |


[参数提取（extract）]: https://httprunner.com/docs/user-guide/enhance-tests/extract/
