---
title: Kubernetes System
date: 2018-06-13 19:29:09
categories:
- devops
tags:
- kubernetes
---
# Cluster

A cluster is a set of physical or virtual machines (or both) combined with other infrastructure resources used by Kubernetes to run your applications.

`kubectl cluster-info`

![Kubernetes Architectural Overview](kubernetes architectural overview.png)
![Kubernetes Architecture](kubernetes architecture.png)

<!-- more -->

# Master

## ETCD

All persistent state data is stored an etcd cluster. This provides a distributed way to store configuration data reliably.

## API server

The API server is the central management point of the entire cluster, and allows the admin to configure Kubernetes workloads and organizational units. The API server is also responsible for making sure etcd and the service details of deployed containers are in agreement.

In other words, the API server validates and configures (on command) the data for pods, services, and replication controllers. It also assigns pods to nodes and synchronizes pod information with service configuration.

## Controller manager

The controller manager service handles the replication processes defined by individual replication tasks. The details of these operations are written to etcd, which the controller manager watches for changes. When a change is seen, the controller manager reads the information and implements the replication procedure that fulfills the desired state, e.g. scaling the application group up or down.

In other words, the controller manager watches etcd for replication tasks and uses the API to enforce the desired state.

## Scheduler

The scheduler assigns workloads to specific nodes in the cluster. It does this by reading in the workload operating requirements, analyzing the current environment (i.e. the health and operational details of the individual nodes in the cluster), and then placing the workload on a suitable node, or nodes.

# Nodes

A node (sometimes called a worker node) is a physical or virtual machine running Kubernetes services, onto which pods can be scheduled.

`kubectl get nodes`

## Docker
A Docker Daemon is running in each node.

## kubelet

The kubelet daemon is the primary agent that runs on each node. The kubelet daemon watches the master API server and ensures the appropriate local containers are started, remain healthy, and continue to run.

## kube-proxy

The kube-proxy daemon runs on each node as a simple network proxy and load balancer for the services on that node.

# References
1. [带着问题学 Kubernetes 架构](https://github.com/jasonGeng88/blog/blob/master/201707/k8s-architecture.md)
2. [带着问题学 Kubernetes 基本单元 Pod](https://github.com/jasonGeng88/blog/blob/master/201707/k8s-pod.md)
3. [带着问题学 Kubernetes 抽象对象 Service](https://github.com/jasonGeng88/blog/blob/master/201707/k8s-service.md)