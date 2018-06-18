---
title: kubectl
date: 2018-06-13 20:51:25
categories:
- Devops
tags:
- Kubernetes
---
# Frequently Used Commands

cd ~/workspace/infrastructure-kubernetes-client-config
./cluster.sh development

k config use-context production

k get namespaces

k get pod -n api-content-production-v1

k logs -n api-content-production-v1 api-content-12-03k

kubectl --namespace=<namespace> exec -it <pod name> sh

k logs -n api-content-production-v1 api-content-12-03k -p

k logs -n api-content-production-v1 api-content-12-03k -f 

k logs [name] | grep p4yvsf --since=14h

kubectl -n=pusher-content-capi-production-v1 logs lqudo-blue-app-1991648162-cnr97 | grep ExportFailure

k get deploy -n api-content-development-v1

k scale deploy -n api-content-development-v1 api-content —replicas=1

k describe pod -n api-content-development-v1 a-edf-pod-name

k get node

k get node —all-namespaces | grep -v Running

k get ing --all-namespaces

kubectl -n loader-content-capi-development-v1 get secrets -o yaml

echo "<encoded_secret_value>" | base64 --decode

In the node:

apk --no-cache add postgresql-client
psql $DATABASE_URL
\dt to list table
\q to quit

psql $DB_URL -t -A -F"," -o <csv_file_name> -c "<sql_query>"

kubectl --n=<namespace> cp <podname>:<file_name_on_pod> <local_file_name>

# References

1. https://kubernetes.io/docs/user-guide/kubectl-cheatsheet/
2. https://github.com/feiskyer/kubernetes-handbook