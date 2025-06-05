---
title: Kubernetes
description: 
published: true
date: 2025-06-05T09:42:06.914Z
tags: 
editor: markdown
dateCreated: 2024-12-04T09:19:29.881Z
---

# Useful Tips
## Get all Docker Registries in use
*run in bash, not zsh! zsh will not recognize the comment correctly*
```bash
kubectl get pods -A -o json |
	jq -r '.items[].spec.containers[].image' |
	sort -nr |
  uniq |
  grep '.*/.*/.*' | # images may omit a registry, in which case we'd incorrectly use the image name as the registry
  cut -d '/' -f 1 |
  uniq
``` 

## Delete broken pods
```bash
kubectl get pod -A |
	grep ContainerStatusUnknown |
	awk '{print "-n "$1" "$2}' |
  xargs -L1 kubectl delete pod
```

## Delete evicted pods
```bash
kubectl get pods -A -o json |
	jq -r '.items[] | select(.status.reason!=null) | select(.status.reason | contains("Evicted")) | "-n \(.metadata.namespace) \(.metadata.name)"' |
  xargs -L1 kubectl delete pod
```

## Refresh GHCR creds
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

# Troubleshooting
## failed to create fsnotify watcher: too many open files
`containerd-shim` uses a ton of inotify user instances. This can be checked using this command (make sure to run as root!):
```sh
for foo in /proc/*/fd/*; do readlink -f $foo; done | grep inotify | cut -d/ -f3 | xargs -I '{}' -- ps --no-headers -o comm {} | sort | uniq -c | sort -nr
```
*Source: https://github.com/k3s-io/k3s/issues/10325#issue-2340098457*

---

To fix this issue, bump the inotify limits:
1. Set these properties in `/etc/sysctl.conf`:
```
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=524288
```
2. run `sysctl -p`
it should print the changed properties (= the two lines above)
3. verify using `sysctl fs.inotify.max_user_instances`

*Source:
https://github.com/k3s-io/k3s/issues/10325#issuecomment-2155008661
https://www.suse.com/support/kb/doc/?id=000020048*