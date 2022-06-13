---
title: "脚本录制"
weight: 5
type: docs
---
创建脚本时，您可以选择脚本录制的方式，来避免大量的脚本手工书写。QuickRunner录制脚本操作简单，可以帮助您简化抓包到脚本编写的过程，大大节省您的测试时间。

具体操作如下：

#### 1. 点击【录制】按钮，启动脚本录制

<img src="/image/QuickRunner/direction/record1.jpeg" alt="QuickRunner" width="500"></br>

#### 2. 在弹窗中添加页面地址，以及配置录制参数。<br/>

 <img src="/image/QuickRunner/direction/record2.jpeg" alt="QuickRunner" width="500">

 ###### 页面请求的处理方式：
      「覆盖当前请求列表」：当录制结束后生成的脚本会覆盖掉当前项目的脚本请求。
      「在当前请求列表后追加」：当录制结束后生成的脚本会追加到当前项目的脚本请求之后。
 ###### 忽略 图片/js/css/字体/视频/音频 资源请求：
        该功能可以帮助您在录制过程中过滤掉 图片/js/css/字体/视频/音频 的资源请求。
 ###### 使用正则表达式过滤：
        该功能可以帮助您在录制过程中按照您所设置的正则表达式，过滤掉的不需要录制的请求。
 ###### 无痕模式：
        在该模式下，录制将忽略浏览器的缓存信息
 ###### 严格模式：
        在该模式下，QuickRunner会自动对录制的请求中存在的 "$" 符号进行处理，避免了存在 "$" 符号的请求会在后续压测中失败。


  #### 3. 录制过程中，可根据您的具体需要添加事务以及集合点。

<img src="/image/QuickRunner/direction/record3.jpeg" alt="QuickRunner" width="800">

  #### 4. 录制结束后，点击【结束录制】结束本次录制，QuickRunner可将您所有的操作录制成脚本。

<img src="/image/QuickRunner/direction/record4.jpeg" alt="QuickRunner" width="800">


>温馨提示：生成脚本后，您可以对脚本按照您的需求进行脚本增强以及修改<br/>
