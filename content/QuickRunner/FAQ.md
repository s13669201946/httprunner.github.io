---
title: "❓常见问题"
weight: 9
description: 关于 QuickRunner 的常见问题
type: docs
---

##### 问：QuickRunner 与 HttpRunner 的关系？

答：quickrunner底层是使用的httprunner，界面更直观，可以通过UI操作生成脚本，操作更方便更简洁

##### 问：QuickRunner 能否部署到无 UI 界面的测试环境中？

答：如果您想借助 QuickRunner 去无 UI 界面的测试环境中压测，那么可以通过命令行的方式启动 QuickRunner 中集成的 HttpRunner，压测结束后，HttpRunner 会将压测结果返回。

##### 问：由于某些特殊情况，请求体中需要添加"$"和我们的引用参数的"$"产生冲突，怎么办？
答：如果body里不需要参数引用，可以将 “ $ ” 替换成“ $$ ”，按照crtl+f可以批量替换。

##### 问：在M1设备上安装应用的时候出现了下图所示的问题，怎么解决？
<img src="/image/QuickRunner/direction/M1.jpg" alt="QuickRunner" width="800">


答：这是由于funppy依赖的grpcio库安装时编译导致的问题，m1设备的用户在第一次使用python自定义函数时，需要添加下面两个环境变量后执行：

```text
export GRPC_PYTHON_BUILD_SYSTEM_OPENSSL=1
export GRPC_PYTHON_BUILD_SYSTEM_ZLIB=1
```