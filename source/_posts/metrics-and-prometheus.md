---
title: Metrics, OpenTracing, Prometheus and Grafana
date: 2018-06-29 23:09:56
categories:
- Microservices
tags:
- Microservices
- Metrics
- DevOps
- Prometheus
- Grafana
- OpenTracing
---
# Calling chain

一个完整的微服务系统包含多个微服务单元，各个微服务子系统存在互相调用的情况，形成一个 调用链。一个客户端请求从发出到被响应 经历了哪些组件、哪些微服务、请求总时长、每个组件所花时长 等信息我们有必要了解和收集，以帮助我们定位性能瓶颈、进行性能调优，因此监控整个微服务架构的调用链十分有必要。

## Zipkin

