---
title: Kubernetes
description: 
published: true
date: 2025-03-03T21:31:31.446Z
tags: 
editor: markdown
dateCreated: 2024-12-04T09:19:29.881Z
---

# Refresh GHCR creds
1. recreate on GitHub
2. backup local dockerconfig
3. `docker login ghcr.io`
4.  ```bash
	kubectl create secret generic ghcr --dry-run=client \
  		--from-file=.dockerconfigjson=$HOME/.config/docker/config.json \
  		--type=kubernetes.io/dockerconfigjson -o yaml > ghcr-pull-secret.local.yml
	```
5. add
```yml
metadata:
	annotations:
		sealedsecrets.bitnami.com/cluster-wide: "true"
```
6. `cat ghcr-pull-secret.local.yml | kubeseal -w ghcr-pull-secret.yml -o yaml` & cleanup resulting file
7. copy to all wanted places