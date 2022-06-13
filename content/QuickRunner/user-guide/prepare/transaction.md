---
title: "事务"
weight: 8
type: docs
---
您可以将一个或多个操作步骤定义为一个事务，可以通俗的理解事务为"人为定义的一系列请求(请求可以是一个或者多个)"。QuickRunner根据事务的开头和结尾标记，计算事务响应时间、成功/失败的事务数、TPS等。
>若某个事务不指定开始，那么该事务的开始位置默认是脚本中的第一条请求。<br/>

>若某个事务不指定结束，那么该事务的结束位置默认是脚本中的最后一条请求。<br/>

该功能的使用方式如下：

<img src="/image/QuickRunner/direction/transaction1.png" alt="QuickRunner" width="800">

<img src="/image/QuickRunner/direction/transaction2.png" alt="QuickRunner" width="800">
<img src="/image/QuickRunner/direction/transaction3.png" alt="QuickRunner" width="800">
<img src="/image/QuickRunner/direction/transaction4.png" alt="QuickRunner" width="800">

