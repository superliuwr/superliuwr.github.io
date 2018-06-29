---
title: Kubernetes Overview
date: 2018-06-12 22:38:21
categories:
- DevOps
tags:
- Kubernetes
---
# Why

Docker -> Docker Compose -> Kubernetes

光靠 Compose 肯定不够，因为 Compose 仅仅解决了应用的描述，运行的问题，但应用的编排，服务注册，服务发现，服务监控，故障恢复，DNS，负载均衡等等功能如何实现？ 

Docker 的作用是计算资源和环境的描述和调度，Kubernetes 的作用是以应用为中心的生命周期的管理。

Kubernetes provides mechanisms for application deployment, scheduling, updating, maintenance, and scaling. A key feature of Kubernetes is that it actively manages the containers to ensure the state of the cluster continually matches the user's intentions.

To provide reliable and scalable distributed system.

Benefits
* immutability
* declarative configuration
* self-healing
* easy scaling
* decoupling applications
* health-checking
* rollouts

Managed KaaS services

<!-- more -->

# Configuration files

Kubernetes defines its constructs through configuration files (called manifests) which can be either YAML or JSON.

# Concepts

## Namespace

A namespace is like a prefix to the name of a resource.

``` yaml
kind: "Namespace"
apiVersion: "v1"
metadata:
  name: "development"
  labels:
    name: "development"
```

* kubectl get namespaces

## Pod

Pods 的定义：Pods 包含一组容器，容器之间共享 namespace，共享存储，共享 IP 地址。

Pods are a collocated group of application containers with shared volumes.

Pods are the smallest deployable units that can be created, scheduled, and managed with Kubernetes.

* kubectl get pods
* kubectl describe pod nginx
* kubectl delete pod nginx

## Replication Controller

Replication Controller 会将集群保持在一个期望的状态。

Replication controllers manage the lifecycle of pods. They insure a specified number of specific pods are running at any given time. They do this by creating or deleting pods as required.

``` yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-nginx
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

* kubectl get rc
* kubectl scale --replicas=3 rc my-nginx

## Service

Service 是一组 Pods 的逻辑集合。在 Service 内部，Kubernetes 会创建负载均衡来将流量代理到可用的 Pods 上。Service 有一个固定的虚拟 IP（ VIP）对外提供服务。
Service 的应用场景在于：假设后端有3个容器实例在运行，并且这3个实例是随时有可能坏掉的（Design for failure），前端的实例无需关注这三个实例各自的状态，而只需要声明它依赖的是后端某一个 Service 即可，让 Replication Controller 自动保证这个后端 Service 的可用性。

Services provide a single stable name and address for a set of pods. They act as basic load balancers.

A service is attached to a replication controller. Each service gets assigned a virtual IP address, which remains constant. As long as we know the service IP address, the service itself will keep track of the pods created by the replication controller, and will distribute requests to them.

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: nginxsvc
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
```

* kubectl get svc nginxsvc

## Label

和名字和 UID 不同，Labels 不提供唯一性，通常，我们期望多个对象使用相同的 Label(s)。通过 Label 的设置和消费，可以轻松的过滤出需要找到的对象

Labels are used to organize and select groups of objects based on key-value pairs.

They are used by every Kubernetes component. For example: the replication controller uses them for service discovery.

## Volumes

A volume is a directory, possibly with some data in it, accessible to a container as part of its filesystem. Volumes are used used, for example, to store stateful app data.

Kubernetes supports several types of volumes:
* emptyDir
* hostPath
* gcePersistentDisk
* awsElasticBlockStore
* nfs
* iscsi
* glusterfs
* rbd
* gitRepo
* secret
* persistentVolumeClaim

## Job

Job负责处理任务，即仅执行一次的任务，它保证批处理任务的一个或多个Pod成功结束

注意Job的RestartPolicy仅支持Never和OnFailure两种，不支持Always

A Job will create a Pod and run the command on it.

``` yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-demo
spec:
  template:
    metadata:
      name: job-demo
    spec:
      restartPolicy: Never
      containers:
      - name: counter
        image: busybox
        command:
        - "bin/sh"
        - "-c"
        - "for i in 9 8 7 6 5 4 3 2 1; do echo $i; done"
```

* kubectl create -f job.yaml
* kubectl get jobs
* kubectl get pods

## CronJob

CronJob则就是在Job上加上了时间调度

``` yaml
apiVersion: batch/v2alpha1
kind: CronJob
metadata:
  name: cronjob-demo
spec:
  successfulJobsHistoryLimit: 10
  failedJobsHistoryLimit: 10
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: hello
            image: busybox
            args:
            - "bin/sh"
            - "-c"
            - "for i in 9 8 7 6 5 4 3 2 1; do echo $i; done"
```

* kubectl create -f cronjob-demo.yaml
* kubectl run hello --schedule="*/1 * * * *" --restart=OnFailure --image=busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster"
* kubectl get cronjob
* kubectl get jobs
* kubectl delete cronjob hello
* Update configmap or secret, need to restart pod
  * kubectl create configmap language --from-literal=LANGUAGE=Chinese -o yaml --dry-run | kubectl replace -f -
  * kubectl create secret generic token --from-literal=TOKEN=bbbbb123456 -o yaml --dry-run | kubectl replace -f -
  * kubectl delete pod -l name=envtest
  * kubectl exec envtest-55d6ff7675-pkwpj -it -- /bin/bash
    * echo $TOKEN
    * echo $LANGUAGE

## ConfigMap & Secret

Secret 和 ConfigMap 之间最大的区别就是 Secret 的数据是用Base64编码混淆过的，不过以后可能还会有其他的差异，对于比较机密的数据（如API密钥）使用 Secret 是一个很好的做法，但是对于一些非私密的数据（比如数据目录）用 ConfigMap 来保存就很好。

``` yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: envtest
  labels:
    name: envtest
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: envtest
    spec:
      containers:
      - name: envtest
        image: cnych/envtest
        ports:
        - containerPort: 5000
        env:
        - name: TOKEN
          valueFrom:
            secretKeyRef:
              name: token
              key: TOKEN
        - name: LANGUAGE
          valueFrom:
            configMapKeyRef:
              name: language
              key: LANGUAGE
```

* kubectl create secret generic token --from-literal=TOKEN=abcd123456000
* kubectl create configmap language --from-literal=LANGUAGE=English
* kubectl get secret
* kubectl get configmap

## ClusterIP & Ingress

### ClusterIP

ClusterIP服务是Kuberntets的默认服务。它在集群内部生成一个服务，供集群内的其他应用访问。外部无法访问。

Use **Kubernetes Proxy** to access cluster internal:

`kubectl proxy --port=8080`

`http://localhost:8080/api/v1/proxy/namespaces/<NAMESPACE>/services/<SERVICE-NAME>:<PORT-NAME>/`

### Ingress

Ingress实际上不是一种服务。相反，它在多个服务前面充当“智能路由”的角色，或者是集群的入口。