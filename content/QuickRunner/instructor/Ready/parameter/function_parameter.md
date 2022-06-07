---
title: "自定义函数"
weight: 4
type: docs
---
当内置函数不满足于您的需求时，您可根据自身的实际需求，自己编写函数脚本。在脚本请求的任意位置调用您自定义的函数，来快速帮您实现各种复杂业务场景的压测脚本的设置。
>通过 ${function} 调用自定义函数<br/>

#### GO语言

 对于Mac版本的，需要自己您自己在环境中安装GO的环境，QuickRunner会自动识别您的Go环境。对于Linux/Windows版本不需要您安装go环境，我们工具会自带go的编译环境，为您省去一部分工作。当您编写完自定义函数后，点击界面左上方的【编译】按钮，等待编译成功后，您就可在脚本中引用您所定义的函数。

该功能的使用方式如下：

<img src="/image/QuickRunner/direction/config-var.jpg" alt="QuickRunner" width="800">
<img src="/image/QuickRunner/direction/function_parameter2.png" alt="QuickRunner" width="800">
<br/>
>温馨提示：Mac可以使用go env验证安装变量，以此来查看go是否安装成功<br/>

#### Python语言
如果您需要在mac上使用该功能，则需要您自己在本地安装好Python环境。
<img src="/image/QuickRunner/direction/python.jpg" alt="QuickRunner" width="800">









