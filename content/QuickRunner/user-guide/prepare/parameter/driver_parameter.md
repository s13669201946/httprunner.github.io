---
title: "参数驱动机制"
weight: 2
type: docs
---
对于不同的请求，当您想对不同的请求中动态变量设置不同的参数时，就需要使用到「参数驱动机制」，该功能通过为不同请求中的动态变量设置不同的参数的方式，来实现请求参数化。
>参数引用使用  $参数名 的形式。<br/>

该功能的使用方式如下：

<img src="/image/QuickRunner/direction/config-var.jpg" alt="QuickRunner" width="700">
<img src="/image/QuickRunner/direction/driver_parameter2.png" alt="QuickRunner" width="700">

>参数选取的方式分为“顺序轮询”，“随机选取”两种形式，系统默认“顺序轮询”方式。<br/>

一般来讲，参数列表中的取值是以“行”为单位的，每一行可以包含多列，循环方式（顺序/随机）作用于该参数列表中的每一行，若您想实现组合参数的效果，需要将不同的参数放入不同的参数列表中。

如下图所示， 有两个参数列表（tableA tableB），其中 tableA 有4组参数，tableB 有2组参数，那么一轮下来，参数组合的数量就是（tableA的参数行数）*（tableB的参数行数），即实现的效果是（ username1,password1,key1 ）、( username1,password1,key2 ）、( username2,password2,key1 ）...... ( username4,password4,key2 ）共6组参数。

<img src="/image/QuickRunner/direction/driver_parameter3.png" alt="QuickRunner1" width="800">

<img src="/image/QuickRunner/direction/driver_parameter4.png" alt="QuickRunner2" width="800">






