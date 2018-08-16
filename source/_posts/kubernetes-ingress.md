---
title: Kubernetes Networking
date: 2018-08-16 21:53:15
categories:
- DevOps
tags:
- Kubernetes
- Traefik
---

## Traditional ways of exposing a service to the external world
### ClusterIP
This is the default. Choosing this value means that you want this service to be reachable only from inside of the cluster.

```yaml
apiVersion: v1
kind: Service
metadata:  
  name: my-internal-service
spec:
  selector:    
    app: my-app
  type: ClusterIP
  ports:  
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
```

Use **Kubernetes Proxy** to access cluster internal:
`kubectl proxy --port=8080`

Then visit `http://localhost:8080/api/v1/proxy/namespaces/<NAMESPACE>/services/<SERVICE-NAME>:<PORT-NAME>/`

### NodePort
A NodePort service is the most primitive way to get external traffic directly to your service. NodePort, as the name implies, opens a specific port on all the Nodes (the VMs), and any traffic that is sent to this port is forwarded to the service.

```yaml
apiVersion: v1
kind: Service
metadata:  
  name: my-nodeport-service
spec:
  selector:    
    app: my-app
  type: NodePort
  ports:  
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30036
    protocol: TCP
```

### LoadBalancer
On top of having a cluster-internal IP and exposing service on a NodePort, also ask the cloud provider for a load balancer which forwards requests to the Service exposed as a :NodePort for each Node. If the cloud provider does not support the feature, the field will be ignored.

## Ingress
Since Kubernetes v1.2.0 you can use Kubernetes ingress which includes support for TLS and L7 http-based traffic routing.

Ingress is actually NOT a type of service. Instead, it sits in front of multiple services and act as a “smart router” or entrypoint into your cluster.

An Ingress is a collection of rules that allow inbound connections to reach the cluster services. It can be configured to give services externally-reachable urls, load balance traffic, terminate SSL, offer name based virtual hosting etc.

In order for the Ingress resource to work, the cluster must have an Ingress controller running. The Ingress controller is responsible for fulfilling the Ingress dynamically by watching the ApiServer’s /ingresses endpoint.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-ingress
spec:
  backend:
    serviceName: other
    servicePort: 8080
  rules:
  - host: foo.mydomain.com
    http:
      paths:
      - backend:
          serviceName: foo
          servicePort: 8080
  - host: mydomain.com
    http:
      paths:
      - path: /bar/*
        backend:
          serviceName: bar
          servicePort: 8080
```

You could go even further, isolate at the infra level where your ingress controller runs and think of it as an “edge router” that enforces the firewall policy for your cluster. This is where Traefik comes to play.

![HA Kubernetes Cluster](k8s_cluster_with_edge_node.png)

## Traefik

Traefik is a modern HTTP reverse proxy and load balancer made to deploy microservices with ease. It supports several backends among Mesos/Marathon and Kubernetes to manage its configuration automatically and dynamically.

We can deploy a Kubernetes cluster similar to the picture above and will run Traefik as DaemonSet.

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: traefik-ingress-controller-v1
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
    kubernetes.io/cluster-service: "true"
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - image: containous/traefik
        name: traefik-ingress-lb
        imagePullPolicy: Always
        ports:
          - containerPort: 80
            hostPort: 80
          - containerPort: 443
            hostPort: 443
          - containerPort: 8080
            hostPort: 8080
        volumeMounts:
          - mountPath: /etc/traefik
            name: traefik-volume
            readOnly: false
        args:
        - --web
        - --kubernetes
        - --configFile=/etc/traefik/traefik.toml
        - --logLevel=DEBUG
      volumes:
        - hostPath:
            path: /etc/traefik
          name: traefik-volume
      nodeSelector:
        role: edge-router
```

Our edge-router will be just another Kubernetes **node** with some restrictions.

We don’t want any other pod to be scheduled to this node so we set **--register-schedulable=false** when running the kubelet as well as giving it a convenient label: **--node-labels=edge-router**.

Kubernetes will run DaemonSets on every node of the cluster even if they are non-schedulable. We only want this DaemonSet to run on the edge-router node so we use “nodeSelector” to match the label we previously added.

```yaml
nodeSelector:
  role: edge-router
```

### configuration

All those rules can be set in .toml file. Rules can also be defined in labels on docker containers and Traefik will pick them up dynamically.

Traditional reverse-proxies require that you configure each route that will connect paths and subdomains to each microservice. In an environment where you add, remove, kill, upgrade, or scale your services many times a day, the task of keeping the routes up to date becomes tedious.

Træfik listens to your service registry/orchestrator API and instantly generates the routes so your microservices are connected to the outside world -- without further intervention from your part.

## Reference
* [Kubernetes NodePort vs LoadBalancer vs Ingress? When should I use what?](https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0)
* [Understanding kubernetes networking: pods](https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727)
* [Understanding kubernetes networking: services](https://medium.com/google-cloud/understanding-kubernetes-networking-services-f0cb48e4cc82)
* [Understanding kubernetes networking: ingress] (https://medium.com/google-cloud/understanding-kubernetes-networking-ingress-1bc341c84078)