---
title: K3s
description: Some tips related to k3s
published: true
date: 2024-05-04T09:38:29.830Z
tags: 
editor: markdown
dateCreated: 2024-05-03T05:59:39.228Z
---

# K3s

## Access embedded etcd
```bash
ETCDCTL_ENDPOINTS='https://127.0.0.1:2379' \
ETCDCTL_CACERT='/var/lib/rancher/k3s/server/tls/etcd/server-ca.crt' \
ETCDCTL_CERT='/var/lib/rancher/k3s/server/tls/etcd/server-client.crt' \
ETCDCTL_KEY='/var/lib/rancher/k3s/server/tls/etcd/server-client.key' \
ETCDCTL_API=3 \
etcdctl member list
```
Source: https://gist.github.com/superseb/0c06164eef5a097c66e810fe91a9d408

## Create user
See [Luc Juggery's article](https://freedium.cfd/https://betterprogramming.pub/k8s-tips-give-access-to-your-clusterwith-a-client-certificate-dfb3b71a76fe)
Make sure to change the `dave` everywhere!