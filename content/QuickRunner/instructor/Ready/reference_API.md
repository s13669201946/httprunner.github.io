---
title: "API引用"
weight: 4
type: docs
---

在接口测试中，除了对单个接口进行测试，还经常涉及多个接口进行联合测试，接口会被复用。为了更好地对接口描述进行管理，QuickRunner 可以使用独立的 JSON/YAML 文件对接口描述进行存储，即每个文件对应一个接口描述。

<img src="/image/QuickRunner/direction/API1.jpeg" alt="QuickRunner" width="800">
<img src="/image/QuickRunner/direction/API2.jpeg" alt="QuickRunner" width="800">

    API引用和用例引用的区别是：
    每个 api 文件和测试用例（testcase）文件的内容基本是相同的。相当于把用例中针对某个 url 接口的测试步骤提取出来，单独存储为一个 JSON/YAML 文件。

>温馨提示：QuickRunner的用例引用只限于当前引用项目下的其他用例。</br>
>具体的JSON/YAML文件的格式可以参考：https://github.com/httprunner/docs里的写法。

