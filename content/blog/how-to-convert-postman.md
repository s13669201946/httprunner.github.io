---
title: HttpRunner v4.0 新增支持 Postman 用例转换
slug: how-to-convert-postman
date: 2022-05-25
---

HttpRunner 从 v4.1.0 开始新增支持 Postman 用例转换功能。Postman 是一款强大、实用且容易上手的接口测试工具，它支持将集合（Collection）导出为 JSON 格式的工程文件，从而可以可以灵活地和其他工具进行协调交互。本文将结合具体案例来介绍使用 HttpRunner 进行 Postman 用例转换的功能。如果使用过程中遇到问题，欢迎在下方评论区留言，我们将根据大家的反馈来进行迭代和优化。

## Postman 工程文件导出

首先，我们基于一个实际的 Postman 项目案例来导出 Postman 的工程文件。在 Postman 的 [Explore 界面]中可以找到非常多的项目案例，这里我们选择一个 Postman 官方提供的工作空间（Workspace）[Published Postman Templates]，打开该工作空间后可以看到官方提供了非常多的集合，这里我们选择其中的一个集合 [Postman Echo] 为例，postman-echo 是一个用来调试 REST 客户端发送请求功能是否正常的服务，具体可以参考 [postman-echo 的接口文档]。

Postman Echo 集合的详情页面如下图所示，通过 `Create a fork` 选项，我们可以将集合复制到自己的工作空间，从而可以进行其他自定义修改。这里我们选择直接通过 `Export` 选项来将 Postman Echo 集合导出为 JSON 格式的工程文件，具体操作方式如下图所示：

![Postman Echo 详情页](/image/postman-echo.png)

在随后弹出的导出选项中，我们选择最新版本 Collection v2.1，点击 Export 将 Postman 工程文件导出到本地。

![Postman Echo Export 选项页](/image/postman-export.png)

读者可以按照上述的操作流程自行导出 Postman 工程文件到本地，此处笔者分享下自己导出的工程文件 [postman_collection.json]，从 JSON 工程文件的形式上可以看出，Postman 导出的工程文件与 HttpRunner 的测试用例有相似之处，都是采用了结构化的方式构建请求样例，不过相比较而言，Postman 的工程文件结构更为复杂。关于该工程文件的详细格式说明和其他细节，感兴趣的读者可以参考 [Collection 格式的介绍文档]，下面我们来演示从 Postman 到 HttpRunner 的用例转换过程。

## Postman 用例转换

使用过 HttpRunner v4.0 的读者可能对 `hrp har2case` 指令不陌生，不过随着后续支持的输入类型越来越多，HttpRunner 的转换功能将不仅仅局限于支持 Postman、Swagger 等工具导出的工程文件到 HttpRunner 测试用例的转换，还会扩展支持 curl、Apach ab 等指令到测试用例的转换，甚至是会支持测试用例的不同形态之间的相互转换，横向的拓展势必会导致指令冗余以及用户的学习成本增加。因此，HttpRunner 之后的用例转换功能将集中在 `hrp convert` 一个指令中，不过仍然兼容之前版本中 `hrp har2case` 的操作，详细规划将在下一期文章《HttpRunner v4.0 用例转换功能规划》中介绍，敬请期待。与 `hrp har2case` 类似，通过 `hrp convert` 可以一次性转换一或多个 Postman 工程文件，单个 Postman 工程文件经过转换会生成对应的单个 JSON/YAML 脚本。下面介绍如何使用 `hrp convert` 指令来进行 Postman 用例转换。

首先介绍 `hrp convert` 各个选项的详细说明：

1. `--to-json/--to-yaml/--to-gotest/--to-pytest` 用于将输入转化为对应形态的测试用例，四个选项中最多只能指定一个，如果不指定则默认会将输入转化为 JSON 形态的测试用例。当前已经支持将 Postman 用例转换为 JSON/YAML/pytest 三种形态的测试用例
2. `--output-dir` 后接测试用例的期望输出目录的路径，用于将转换生成的测试用例输出到对应的文件夹，需要注意该路径必须是存在且合法的
3. `--profile` 后接 profile 配置文件的路径，profile 文件的后缀可以为 .json/.yaml/.yml，目前支持修改输入中的 Headers 和 Cookies 信息，并且支持替换（不存在则会创建）以及覆盖两种修改模式，下面给出这两种修改模式的 profile 配置文件示例：

- 根据 profile 替换指定的 Headers 和 Cookies 信息

```yaml
headers:
  Header1: "this header will be created or updated"
cookies:
  Cookie1: "this cookie will be created or updated"
```

- 根据 profile 覆盖原有的 Headers 和 Cookies 信息

```yaml
override: true
headers:
  Header1: "all original headers will be overridden"
cookies:
  Cookie1: "all original cookies will be overridden"
```

下面执行 `hrp convert` 指令来进行 Postman 用例转换。在前面的 Postman 工程文件导出步骤中，我们得到了文件 postman_collection.json，在该文件所在的目录打开终端并执行 `hrp convert postman_collection.json --to-yaml` 即可实现转换过程，此处以转化为 YAML 测试用例为例，执行结果如下：

```shell
$ hrp convert postman_collection.json --to-yaml                 
11:53AM INF Set log to color console other than JSON format.
11:53AM ??? Set log level
11:53AM INF load file path=postman_collection.json
11:53AM INF load file path=postman_collection.json
11:53AM INF load case as: postman input path=postman_collection.json
11:53AM INF start converting input path=postman_collection.json
11:53AM INF convert teststep method=GET url=https://postman-echo.com/get?foo1=bar1&foo2=bar2
11:53AM INF convert teststep method=POST url=https://postman-echo.com/post
11:53AM INF convert teststep method=POST url=https://postman-echo.com/post
11:53AM INF convert teststep method=PUT url=https://postman-echo.com/put
11:53AM INF convert teststep method=PATCH url=https://postman-echo.com/patch
11:53AM INF convert teststep method=DELETE url=https://postman-echo.com/delete
11:53AM INF convert teststep method=GET url=https://postman-echo.com/headers
11:53AM INF convert teststep method=GET url=https://postman-echo.com/response-headers?foo1=bar1&foo2=bar2
11:53AM INF convert teststep method=GET url=https://postman-echo.com/basic-auth
11:53AM INF convert teststep method=GET url=https://postman-echo.com/digest-auth
11:53AM INF convert teststep method=GET url=https://postman-echo.com/auth/hawk
11:53AM INF convert teststep method=GET url=https://postman-echo.com/oauth1
11:53AM INF convert teststep method=GET url=https://postman-echo.com/cookies/set?foo1=bar1&foo2=bar2
11:53AM INF convert teststep method=GET url=https://postman-echo.com/cookies
11:53AM INF convert teststep method=GET url=https://postman-echo.com/cookies/delete?foo1&foo2
11:53AM INF convert teststep method=GET url=https://postman-echo.com/status/200
11:53AM INF convert teststep method=GET url=https://postman-echo.com/stream/5
11:53AM INF convert teststep method=GET url=https://postman-echo.com/delay/2
11:53AM INF convert teststep method=GET url=https://postman-echo.com/encoding/utf8
11:53AM INF convert teststep method=GET url=https://postman-echo.com/gzip
11:53AM INF convert teststep method=GET url=https://postman-echo.com/deflate
11:53AM INF convert teststep method=GET url=https://postman-echo.com/ip
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/now
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/valid?timestamp=2016-10-10
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/format?timestamp=2016-10-10&format=mm
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/unit?timestamp=2016-10-10&unit=day
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/add?timestamp=2016-10-10&years=100
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/subtract?timestamp=2016-10-10&years=50
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/start?timestamp=2016-10-10&unit=month
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/object?timestamp=2016-10-10
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/before?timestamp=2016-10-10&target=2017-10-10
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/after?timestamp=2016-10-10&target=2017-10-10
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/between?timestamp=2016-10-10&start=2017-10-10&end=2019-10-10
11:53AM INF convert teststep method=GET url=https://postman-echo.com/time/leap?timestamp=2016-10-10
11:53AM INF convert teststep method=GET url=https://postman-echo.com/digest-auth
11:53AM INF dump data to yaml path=/Users/bytedance/code/go/httprunner/postman_collection_test.yaml
11:53AM INF conversion completed output files=["postman_collection_test.yaml"]
```

在执行了上述操作后，可以得到转换生成的 HttpRunner YAML 格式的测试用例，之后可以使用 `hrp run` 和 `hrp boom` 基于该测试用例来进行后续的接口/性能测试。同理，我们也可以指定 `--to-pytest` 选项来将输入转化为 pytest 脚本形态的测试用例：

```python
# NOTE: Generated By HttpRunner v4.1.0-beta
# FROM: postman_collection_test.json


from httprunner import HttpRunner, Config, Step, RunRequest, RunTestCase

class TestCasePostmanCollectionTest(HttpRunner):

    config = Config("Postman Echo")

    teststeps = [
        Step(
            RunRequest("Request Methods - GET Request")
            .get("https://postman-echo.com/get")
            .with_params(**{"foo1": "bar1", "foo2": "bar2"})
        ),
        Step(
            RunRequest("Request Methods - POST Raw Text")
            .post("https://postman-echo.com/post")
            .with_headers(**{"Content-Type": "text/plain"})
            .with_data("This is expected to be sent back as part of response body.")
        ),
        Step(
            RunRequest("Request Methods - POST Form Data")
            .post("https://postman-echo.com/post")
            .with_headers(**{"Content-Type": "application/x-www-form-urlencoded"})
            .with_data({"foo1": "bar1", "foo2": "bar2"})
        ),
        Step(
            RunRequest("Request Methods - PUT Request")
            .put("https://postman-echo.com/put")
            .with_headers(**{"Content-Type": "text/plain"})
            .with_data("This is expected to be sent back as part of response body.")
        ),
        Step(
            RunRequest("Request Methods - PATCH Request")
            .patch("https://postman-echo.com/patch")
            .with_headers(**{"Content-Type": "text/plain"})
            .with_data("This is expected to be sent back as part of response body.")
        ),
        Step(
            RunRequest("Request Methods - DELETE Request")
            .delete("https://postman-echo.com/delete")
            .with_headers(**{"Content-Type": "text/plain"})
            .with_data("This is expected to be sent back as part of response body.")
        ),
        Step(
            RunRequest("Headers - Request Headers")
            .get("https://postman-echo.com/headers")
            .with_headers(**{"my-sample-header": "Lorem ipsum dolor sit amet"})
        ),
        Step(
            RunRequest("Headers - Response Headers")
            .get("https://postman-echo.com/response-headers")
            .with_params(**{"foo1": "bar1", "foo2": "bar2"})
        ),
        Step(
            RunRequest("Authentication Methods - Basic Auth").get(
                "https://postman-echo.com/basic-auth"
            )
        ),
        Step(
            RunRequest("Authentication Methods - DigestAuth Success")
            .get("https://postman-echo.com/digest-auth")
            .with_headers(
                **{
                    "Authorization": 'Digest username="postman", realm="Users", nonce="ni1LiL0O37PRRhofWdCLmwFsnEtH1lew", uri="/digest-auth", response="254679099562cf07df9b6f5d8d15db44", opaque=""'
                }
            )
        ),
        Step(
            RunRequest("Authentication Methods - Hawk Auth").get(
                "https://postman-echo.com/auth/hawk"
            )
        ),
        Step(
            RunRequest("Authentication Methods - OAuth1.0 (verify signature)").get(
                "https://postman-echo.com/oauth1"
            )
        ),
        Step(
            RunRequest("Cookie Manipulation - Set Cookies")
            .get("https://postman-echo.com/cookies/set")
            .with_params(**{"foo1": "bar1", "foo2": "bar2"})
        ),
        Step(
            RunRequest("Cookie Manipulation - Get Cookies").get(
                "https://postman-echo.com/cookies"
            )
        ),
        Step(
            RunRequest("Cookie Manipulation - Delete Cookies")
            .get("https://postman-echo.com/cookies/delete")
            .with_params(**{"foo1": "", "foo2": ""})
        ),
        Step(
            RunRequest("Utilities - Response Status Code").get(
                "https://postman-echo.com/status/200"
            )
        ),
        Step(
            RunRequest("Utilities - Streamed Response").get(
                "https://postman-echo.com/stream/5"
            )
        ),
        Step(
            RunRequest("Utilities - Delay Response").get(
                "https://postman-echo.com/delay/2"
            )
        ),
        Step(
            RunRequest("Utilities - Get UTF8 Encoded Response").get(
                "https://postman-echo.com/encoding/utf8"
            )
        ),
        Step(
            RunRequest("Utilities - GZip Compressed Response").get(
                "https://postman-echo.com/gzip"
            )
        ),
        Step(
            RunRequest("Utilities - Deflate Compressed Response").get(
                "https://postman-echo.com/deflate"
            )
        ),
        Step(
            RunRequest("Utilities - IP address in JSON format").get(
                "https://postman-echo.com/ip"
            )
        ),
        Step(
            RunRequest("Utilities / Date and Time - Current UTC time").get(
                "https://postman-echo.com/time/now"
            )
        ),
        Step(
            RunRequest("Utilities / Date and Time - Timestamp validity")
            .get("https://postman-echo.com/time/valid")
            .with_params(**{"timestamp": "2016-10-10"})
        ),
        Step(
            RunRequest("Utilities / Date and Time - Format timestamp")
            .get("https://postman-echo.com/time/format")
            .with_params(**{"format": "mm", "timestamp": "2016-10-10"})
        ),
        Step(
            RunRequest("Utilities / Date and Time - Extract timestamp unit")
            .get("https://postman-echo.com/time/unit")
            .with_params(**{"timestamp": "2016-10-10", "unit": "day"})
        ),
        Step(
            RunRequest("Utilities / Date and Time - Time addition")
            .get("https://postman-echo.com/time/add")
            .with_params(**{"timestamp": "2016-10-10", "years": "100"})
        ),
        Step(
            RunRequest("Utilities / Date and Time - Time subtraction")
            .get("https://postman-echo.com/time/subtract")
            .with_params(**{"timestamp": "2016-10-10", "years": "50"})
        ),
        Step(
            RunRequest("Utilities / Date and Time - Start of time")
            .get("https://postman-echo.com/time/start")
            .with_params(**{"timestamp": "2016-10-10", "unit": "month"})
        ),
        Step(
            RunRequest("Utilities / Date and Time - Object representation")
            .get("https://postman-echo.com/time/object")
            .with_params(**{"timestamp": "2016-10-10"})
        ),
        Step(
            RunRequest("Utilities / Date and Time - Before comparisons")
            .get("https://postman-echo.com/time/before")
            .with_params(**{"target": "2017-10-10", "timestamp": "2016-10-10"})
        ),
        Step(
            RunRequest("Utilities / Date and Time - After comparisons")
            .get("https://postman-echo.com/time/after")
            .with_params(**{"target": "2017-10-10", "timestamp": "2016-10-10"})
        ),
        Step(
            RunRequest("Utilities / Date and Time - Between timestamps")
            .get("https://postman-echo.com/time/between")
            .with_params(
                **{
                    "end": "2019-10-10",
                    "start": "2017-10-10",
                    "timestamp": "2016-10-10",
                }
            )
        ),
        Step(
            RunRequest("Utilities / Date and Time - Leap year check")
            .get("https://postman-echo.com/time/leap")
            .with_params(**{"timestamp": "2016-10-10"})
        ),
        Step(
            RunRequest("Auth: Digest - DigestAuth Request").get(
                "https://postman-echo.com/digest-auth"
            )
        ),
    ]


if __name__ == "__main__":
TestCasePostmanCollectionTest().test_start()
```

## 注意事项

1. `hrp convert` 可以自动识别输入类型，因此不需要通过选项来手动制定输入类型，如遇到无法识别、不支持或转换失败的情况，则会输出错误日志并跳过，不会影响其他转换过程的正常进行
2. 输出的测试用例文件名格式为 `Postman 工程文件名称（不带拓展名）` + `_test` + `.json/.yaml/.go/.py 后缀` ，如果该文件已经存在则会进行覆盖
3. 在 profile 配置文件中，指定 `override` 字段为 `false/true` 可以选择修改模式为替换/覆盖，需要注意的是，如果不指定该字段则默认修改模式为替换模式
4. 如果 Postman 工程文件中包含了文件夹层级结构，则 `hrp convert` 会将层级结构递归展开，所有请求按顺序转换生成 HttpRunner 测试用例中的若干步骤，步骤名称为该请求在原始 Postman 工程文件中的路径，路径中的层级之间用连字符“-”分隔，从而可以有效帮助用户定位到各个请求在原始 Postman 工程文件中的位置
5. 完整支持 Postman 请求 Params 中的请求参数变量、请求路径变量机制，`hrp convert` 会自动解析各个变量名并进行替换
6. 完整支持 Postman 请求 Body 中的 form-data/x-www-form-urlencoded/raw 类型，不过暂不支持 binary / GraphQL 类型

## What's next

1. 支持转换 Postman 请求 Authorization 中的内容，补充测试用例 Headers 信息
2. 支持转换 Postman 请求 Pre-request Script 中的内容为 Setup 操作
3. 支持转换 Postman 请求 Tests 中的内容为请求结果断言操作

> 本文作者：卜卜星（HttpRunner 核心开发者）<br/>
> HttpRunner 项目官网: https://httprunner.com/<br/>
> 如果 HttpRunner 对你有过帮助，麻烦帮忙给个 ⭐️star⭐️ 鼓励下吧<br/>
> https://github.com/httprunner/httprunner

[Explore 界面]: https://www.postman.com/explore
[Published Postman Templates]: https://www.postman.com/postman/workspace/published-postman-templates/overview
[Postman Echo]: https://www.postman.com/postman/workspace/published-postman-templates/collection/631643-f695cab7-6878-eb55-7943-ad88e1ccfd65
[postman-echo 的接口文档]: https://www.postman.com/postman/workspace/published-postman-templates/documentation/631643-f695cab7-6878-eb55-7943-ad88e1ccfd65
[postman_collection.json]: https://www.postman.com/collections/92a0446c7162b800282f
[Collection 格式的介绍文档]: https://blog.postman.com/travelogue-of-postman-collection-format-v2/
