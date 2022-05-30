---
title: "动态运算逻辑"
weight: 3
type: docs
---
QuickRunner支持动态运算，您可以在请求的任意位置调用QuickRunner的内置函数，以便帮助您快速创建压测脚本。
>采用  ${函数名} 的方式调用内置函数。<br/>

目前QuickRunner所支持的函数有：

| 函数名         | 参数     | 函数说明               |
| ----------------- | ---------- | -------------------------- |
| get_timestamp     | ()         | 获取当前时间的十三位时间戳 |
| gen_random_string | (n int)    | 获取 n 位随机字符串 |
| max               | (m,n int)  | 获取m,n中的最大值   |
| md5               | (s string) | 获取输入字符串的MD5值 |


该功能的使用方式如下：

<img src="/image/QuickRunner/direction/calculate_parameter.png" alt="QuickRunner" width="800">









