---
title: "集合点"
weight: 7
type: docs
---
集合点用于实现规定数量的虚拟用户能在同一时刻同时执行某条任务的功能。在测试计划中，您可能要求被测系统能够承受多人同时提交数据，那么，您可在提交数据操作前面加入「集合点」来达到您想要的效果。

目前QuickRunner可以通过设置“指定虚拟用户数”、“全部用户”、“虚拟用户百分数”三种形式来指定需要等待的用户数；当虚拟用户运行到集合点时，QuickRunner 就会检查有多少用户到达到集合点，如果数量达不到指定数量的用户数，QuickRunner 就会命令已经到集合点的用户在此等待，只有当在集合点等待的用户数量达到指定的数量时，QuickRunner才释放集合点，让这些用户线程继续往下执行，从而达到测试计划中的目标。

另外，QuickRunner也支持设置超时时间。如果用户到达集合点的时间间隔超过指定的超时时间后，QuickRunner就会释放所有等待的用户线程，让它们继续向下执行，系统默认的超时时间是5秒。

该功能的使用方式如下：

<img src="/image/QuickRunner/direction/collection_point1.png" alt="QuickRunner" width="800">

<img src="/image/QuickRunner/direction/collection_point2.png" alt="QuickRunner" width="800">

