---
title: 版本号机制
weight: 1
description: HttpRunner 的版本号机制规范
---

作为一个开源的基础框架，版本号是至关重要的。

从 2.0 版本开始，HttpRunner 采用 [`Semantic Versioning`][SemVer] 机制进行版本号管理，即 `MAJOR.MINOR.PATCH` 的形式。

- MAJOR: 重大版本升级并出现前后版本不兼容时加 1
- MINOR: 大版本内新增功能并且保持版本内兼容性时加 1
- PATCH: 功能迭代过程中进行问题修复（bugfix）时加 1

在遵循如上主体原则的前提下，也会根据需要，在版本号后面添加先行版本号（-alpha/beta）作为延伸，例如 `v4.0.0-beta`。

[SemVer]: https://semver.org/