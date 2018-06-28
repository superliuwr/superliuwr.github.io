---
title: CI, CD and Pipelines
date: 2018-06-28 22:49:17
categories:
- DevOps
tags:
- CI
- CD
---
# What are CI and CD

Continuous Integration (CI) is a development practice that requires developers to integrate code into a shared repository several times a day. Each check-in is then verified by an automated build, allowing teams to detect problems early.

Continuous Delivery (CD) is the natural extension of Continuous Integration: an approach in which teams ensure that every change to the system is releasable, and that we can release any version at the push of a button. Continuous Delivery aims to make releases boring, so we can deliver frequently and get fast feedback on what users care about.

# 持续集成应有的标准规范

1. 频繁提交
2. 自动化测试
3. 较短的构建和测试过程
4. 本地开发环境与持续集成环境、测试环境、生产环境一致

# Tools

* Github/Gitlab/Bitbucket
* Buildkite/Jenkins/Codefresh/Codeship
* Docker

# 发布策略

Canary

# Pipeline

