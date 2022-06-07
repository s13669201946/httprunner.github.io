---
title: "检测被测系统"
weight: 2
type: docs
description: 自动检测被测服务所在的系统信息。
---
QuickRunner提供了「检测被测系统」功能，当勾选【检测被测系统】框时，QuickRunner会自动检测被测服务所在的系统信息。

该功能的使用方式如下：
###  1. 下载对应系统的quickrunner_system_agent二进制文件

在 [GitHub Releases][releases] 页面中选择您需要的版本,当前 QuickRunner 支持的环境包括：

- macOS(darwin) + amd64(x86)
- macOS(darwin) + arm64(M1)
- linux + amd64(x86)
- linux + arm64
- windows + amd64(x86)

如果你的系统环境不在这个范围内，那么你只能自行拉取源码进行编译。

###  2. 执行本地二进制文件，启动quickrunner_system_agent
```bash
# Linux/Mac执行指令格式：
$ ./二进制文件路径 --pull.disable-default --pushgateway.listen-address="quickrunner_data_bus地址:端口号"  --ip="quickrunner_system_agent地址" --collector.disable-defaults --collector.cpu --collector.meminfo --collector.diskstats --collector.netdev
# Windows执行指令:
$ 二进制exe文件路径  --pull.disable-default --pushgateway.listen-address="quickrunner_data_bus地址:端口号" --ip="quickrunner_system_agent地址" --collectors.enabled="cpu,os,cs,logical_disk,net"

#参数解析：
# quickrunner_data_bus地址：一般为客户端所在的IP，端口号为9091。如果存在多个地址使用|分割，如："127.0.0.1:9091|192.168.1.2:9091"。该参数用于给客户端上报数据。
# quickrunner_system_agent地址：指定当前quickrunner_system_agent所部署设备的ip地址，如果未设置，默认第一个非回环ipv4地址。

#举例：
# linux/mac：./quickrunner_system_agent-darwin-amd64 --pull.disable-default --pushgateway.listen-address="127.0.0.1:5331"  --ip=127.0.0.1 --collector.disable-defaults --collector.cpu --collector.meminfo --collector.diskstats --collector.netdev

# windows: quickrunner_system_agent.exe  --pull.disable-default --pushgateway.listen-address="127.0.0.1:5331" --ip=127.0.0.1 --collectors.enabled="cpu,os,cs,logical_disk,net"
```
>温馨提示：如果执行指令的过程中报权限不够，请先修改一下文件权限，再执行。

###  3. 打开客户端，勾选「检测被测系统」框，开启QuickRunner检测
<img src="/image/QuickRunner/direction/device_info.png" alt="QuickRunner" width="800">

[releases]: https://github.com/s13669201946/QuickRunner/releases/tag/QuickrunnerV1.0.0
[github-actions]:https://github.com/s13669201946/QuickRunner/releases/tag/QuickrunnerV1.0.0