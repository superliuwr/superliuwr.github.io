---
title: Helm
date: 2018-06-11 23:04:25
categories: devops
tags:
- helm
- devops
- kubernetes
---
# Introduction

Helm is a package manager for Kubernetes applications.

Helm packages all of the different Kubernetes resources (such as deployments, services, and ingress) into a chart, which may be hosted in a repository.

Users can pull down charts and install them on any number of Kubernetes clusters.

Helm’s approach scales from monoliths to complex micro service applications. 

Helm itself uses a client-server model. The helm command (the client) talks to the tiller  (the server). The Helm client may interact with any number of different tiller services. In practice, there is a single tiller service running one Kubernetes cluster. This helps teams collaborate. It also means that Helm may run anywhere, such as on your CI servers or on your own computer. Tiller does the work to coordinate with Kubernetes and get the chart installed.

<!-- more -->

# Install Helm

Run following to create a service account for Tiller in GCE as Kubernetes enables RBAC since version 1.8:

* kubectl create serviceaccount --namespace kube-system tiller
* kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
* kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'      
* helm init --service-account tiller --upgrade

`brew install kubernetes-helm`

`helm init`:
This will install Tiller in the cluster defined by your context according to your Kubectl configuration.

To check if Tiller is running:
`kubectl -n kube-system get pods|grep tiller`


# Frequently Used Commands

```
helm repo update

helm install stable/mysql

helm ls

helm delete smiling-penguin

helm status smiling-penguin

helm get -h
```


# Using Helm

## Three big concepts

* A Chart is a Helm package. It contains all of the resource definitions necessary to run an application, tool, or service inside of a Kubernetes cluster. Think of it like the Kubernetes equivalent of a Homebrew formula, an Apt dpkg, or a Yum RPM file.
* A Repository is the place where charts can be collected and shared. It is like Perl’s CPAN archive or the Fedora Package Database, but for Kubernetes packages.
* A Release is an instance of a chart running in a Kubernetes cluster. One chart can often be installed many times into the same cluster. And each time it is installed, a new release is created. Consider a MySQL chart. If you want two databases running in your cluster, you can install that chart twice. Each one will have its own release, which will in turn have its own release name.

With these concepts in mind, we can now explain Helm like this:

* Helm installs charts into Kubernetes, creating a new release for each installation. And to find new charts, you can search Helm chart repositories.


# Usage

To find predefined charts:

* helm search
* helm search mysql

To check a chart:

* helm inspect stable/mariadb

To install dependencies of a chart:

* helm dep up foochart

To install a chart with default config:

* helm install stable/mariadb

To check status of a release:

* helm status happy-panda

To see what options are configurable in a chart:

* helm inspect values stable/mariadb

Then override some config then install:

* helm install -f config.yaml stable/mariadb

Update a release:

* helm upgrade -f panda.yaml happy-panda stable/mariadb

Check custom config:

* helm get values happy-panda

Roll back a release to given revision:

* helm rollback happy-panda 1

Uninstall a release:

* helm delete happy-panda

Show releases:

* helm list —all
* helm list —deleted

Working with Repos:

* helm repo list
* helm repo add dev https://example.com/dev-charts


Creating and installing charts:

* helm create deis-workflow
* helm lint
* helm package deis-workflow
* helm install --dry-run --debug <chart_dir>
* helm install ./deis-workflow-0.1.0.tgz

After installation:

* export POD_NAME=$(kubectl get pods —namespace default -l "app=hello-helm,release=kilted-bobcat" -o jsonpath="{.items[0].metadata.name}")
* kubectl port-forward $POD_NAME 8080:80

Then you can now access the service you just installed by visiting http://127.0.0.1:8080


# Charts

A chart contains:

* Chart.yml that describes the chart (this includes the name, description, and version information that we saw in helm search)
* One or more Kubernetes manifest templates (deployment, pod, service, etc.)
* An optional third item: values.yml. This file declares the default values used at the time of installation. These may be overridden at installation via the —set or -f flags

Inside of this directory, Helm will expect a structure that matches this:

```
wordpress/
  Chart.yaml          # A YAML file containing information about the chart
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  requirements.yaml   # OPTIONAL: A YAML file listing dependencies for the chart
  values.yaml         # The default configuration values for this chart
  charts/             # A directory containing any charts upon which this chart depends.
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```

Helm Chart templates are written in the Go template language, with the addition of 50 or so add-on template functions from the Sprig library and a few other specialized functions.

All template files are stored in a chart’s templates/ folder. When Helm renders the charts, it will pass every file in that directory through the template engine.

Values that are supplied via a values.yaml file (or via the --set flag) are accessible from the .Values object in a template.

Pre-defined values: .Values, Chart, Release, etc.


# CI/CD
To be added

# References
1. https://www.influxdata.com/blog/packaged-kubernetes-deployments-writing-helm-chart/
