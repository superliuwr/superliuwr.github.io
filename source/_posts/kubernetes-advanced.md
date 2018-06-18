---
title: Kubernetes Advanced
date: 2018-06-13 19:27:32
categories:
- Devops
tags:
- Kubernetes
---
# Service Discovery

Kubernetes 里服务发现有多种方式：

1. 通过环境变量注入发现服务。
当 Pods 在集群的 Node 中运行时，Kubernetes 会为它增加一些列的环境变量。

2. 通过 Kubernetes DNS 服务器。
Kubernetes DNS 服务器会订阅 Kubernetes API创建 Service 的事件，并且为每个 Service 记录一个 DNS 的记录。这样做的好处是：只要 DNS 服务在集群里有访问权限，那么所有的 Pods 都能访问新注册的 Service。举例：如果你有一个 Service 在 Kubernetes Namespace "my-ns"下 ，名字叫做叫 "my-service"，那么一条叫做"my-service.my-ns"的记录就会被创建。所有在"my-ns" Namespace 里的 Pods 都能通过 DNS 查找到"my-service"。

<!-- more -->

# Resource Monitoring

# Health Check

# Horizontal Auto Scaling

# Networking

# Rolling Deployment and Rollback

A deployment object holds one or more replica sets to support the rollback mechanism. In other words, it creates a new replica set every time the deployment configuration is changed and keeps the previous version in order to have the option of rollback. Only one replica set will be in active state at a certain time.