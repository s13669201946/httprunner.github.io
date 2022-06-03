---
title: 【MTSC 2022 年度开源项目评选】HttpRunner 期待你的支持
slug: mtsc-2022-opensource
date: 2022-06-02
---

近期，MTSC 大会委员会联合 TesterHome 社区，发起了 MTSC 2022 年度最佳开源项目[评选活动][9]。经过组委会初审，最终有包括 HttpRunner 在内的 [17 个开源项目][10]通过初审入围评选。

对于 HttpRunner 开源项目，我们团队投入了非常多的时间和精力，我们也期望能通过这次活动，提升 HttpRunner 开源项目的知名度，为 HttpRunner 更长远的发展打好基础。

**经过与活动主办方沟通确认**，我们可以对 HttpRunner 做适当的宣传，欢迎认可和看好 HttpRunner 的朋友能给 HttpRunner 投上一票。

接下来，本文将主要展示 HttpRunner 开源项目的一些核心数据。

## 项目投入情况

在 2017 年 6 月份的时候我写了一篇博客，[《接口自动化测试的最佳工程实践（ApiTestEngine）》][1]，并同时开始了 ApiTestEngine（HttpRunner 的前身）的开发工作。令我自己也没有想到的是，HttpRunner 居然从最开始的个人业余练手项目一路迭代至今，马上就满整整 5 年了。

在这 5 年期间，HttpRunner 先后：

- 发布了 4 个大版本（v1~v4）和 [99 个小版本][2]
- 共有 [22 位开发者][3]参与了代码贡献，提交了 [2839 次 commit][4]，代码变更量 [22w+][3]
- 编写了 30+ 篇技术博客，在 [TesterHome][TesterHome] 上有 8 篇相关文章被评选为精华帖

![](/image/httprunner-commits.png)

说实话，HttpRunner 能持续维护这么多年的确是非常不容易的。在这些冰冷的数字背后，凝结了 HttpRunner 开发者和用户们太多的心血。

## 项目核心用户数据

值得欣慰的是，当前 HttpRunner 已经有了一定的用户基数和知名度，在搜索引擎和各种主流技术社区搜索 HttpRunner 都能看到一些用户自发分享的文章，甚至还有培训班以此开设了付费课程，以及有人写书时做了较大篇幅的介绍。这些反馈给了我们极大的鼓舞，让我们有更大的动力将 HttpRunner 变得更好。

如下是关于 HttpRunner 的一些核心用户数据，分为`客观数据`和`主观评价`两部分。

### 使用情况（客观数据）

HttpRunner 在 GitHub 上的 star 数稳步增长，截至今日累积收获了 [2963 个 star][5] 和 [1072 个 fork][6]。

![](/image/star-history-220602.png)

在 pypi 下载统计方面，Python 版的 HttpRunner [累积下载 668,524 次][8]，近 30 天 11,442 次，近 7 天 3,002 次。

[![Downloads](https://pepy.tech/badge/httprunner)](https://pepy.tech/project/httprunner)
[![Downloads](https://pepy.tech/badge/httprunner/month)](https://pepy.tech/project/httprunner)
[![Downloads](https://pepy.tech/badge/httprunner/week)](https://pepy.tech/project/httprunner)

在 GitHub 项目访问量方面，最近 14 天（5.20~6.02）累积有 [6231 次仓库下载][7] 和 [10998 次页面浏览][7]。

### 用户反馈（主观评价）

除了客观的用户使用数据之外，HttpRunner 也收到了用户较多的正面反馈。

在 MTSC 大会方面，HttpRunner 之前在大会上做过两次分享：

- MTSC 2018 为 HttpRunner 颁发了 [2018 年度开源贡献奖][TTF-2018]
- MTSC 2018 分享议题 [大疆互联网的一站式自动化测试解决方案（基于HttpRunner）][MTSC2018-PPT]：被评选为服务端测试专场 11 个议题中的[最佳议题][MTSC2018-score]。
- MTSC 2019 分享议题 [HttpRunner 2.0 技术架构与接口测试应用][MTSC2019-PPT]：被评选为 TTF&质量保障2 专场 10 个议题中的[最佳议题][MTSC2019-score]。

在 2022 年 2 月，基于首轮面向 HttpRunner 用户的 252 份调研问卷，HttpRunner 整体用户满意度平均分 4.3 分，并收获了非常多正面的评价，详见 [HttpRunner 首轮用户调研报告（2022.02）][user-survey-report]。

## 总结

以上便是 HttpRunner 开源项目 5 年来的投入情况和核心用户数据汇总。感谢大家一直以来的关注和支持，HttpRunner 将持续进行运营和迭代，期待能为大家的工作提供更多的助力。

最后，本次 MTSC 开源项目评选活动，[HttpRunner] 期待你的支持和鼓励。

- 点击[【投票链接🔗】][11]，为 [HttpRunner] 投上一票吧 🚀
- 截止日期：2022 年 6 月 30 日

再次感谢大家！🌹

[1]: https://debugtalk.com/post/ApiTestEngine-api-test-best-practice/
[2]: https://github.com/httprunner/httprunner/tags
[3]: https://github.com/httprunner/httprunner/graphs/contributors
[4]: https://github.com/httprunner/httprunner/commits/master
[5]: https://github.com/httprunner/httprunner/stargazers
[6]: https://github.com/httprunner/httprunner/network/members
[7]: https://github.com/httprunner/httprunner/graphs/traffic
[8]: https://pepy.tech/project/httprunner
[9]: https://testerhome.com/topics/33054
[10]: https://testerhome.com/topics/33330
[11]: https://tp.wjx.top/vj/h1AEk9i.aspx
[MTSC2018-PPT]: https://github.com/debugtalk/speech/blob/master/DJI-HttpRunner.pdf
[MTSC2018-score]: https://testerhome.com/topics/15163
[MTSC2019-PPT]: https://github.com/debugtalk/speech/blob/master/MTSC2019-HttpRunner-2.0.pdf
[MTSC2019-score]: https://testerhome.com/topics/20059
[TesterHome]: https://testerhome.com/debugtalk/topics
[TTF-2018]: https://testerhome.com/topics/15077
[user-survey-report]: https://httprunner.com/blog/user-survey-report
[HttpRunner]: https://github.com/httprunner/httprunner
