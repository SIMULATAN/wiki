---
title: K3s
description: Some tips related to k3s
published: true
date: 2025-07-31T13:20:40.564Z
tags: 
editor: markdown
dateCreated: 2024-05-03T05:59:39.228Z
---

# K3s
## Setup Command
You'll need a K3s Bootstrap token. You can probably use [`k3s token create`](https://docs.k3s.io/cli/token#k3s-token-1) for this purpose, although I cannot remember how I originally joined the nodes.
`/var/lib/rancher/k3s/server/token`'s last part could have something to do with it too.
```bash
curl -sfL https://get.k3s.io | K3S_TOKEN='...' sh -s - server \
    --cluster-init \
    --write-kubeconfig-mode=644 \
    --tls-san=`ip addr | grep '172.' | head -1 | awk '{$1=$1;print}' | cut -d ' ' -f 2 | cut -d '/' -f 1` # Optional, needed if using a fixed registration address
```

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

## OIDC
### Setup Authelia
Add an OIDC client to Authelia. The following shows an example using the official Helm chart. Of course, you can simply add the client by any other means.
The `client_secret` approach can be found in my k8s-ops repo: https://github.com/SIMULATAN/k8s-ops/tree/main/auth/authelia
```yml
configMap:
  identity_providers:
    oidc:
      clients:
        - client_id: k8s-main
          client_name: Kubernetes Main
          client_secret:
          path: /secrets/authelia-oidc-secrets/CLIENT_SECRET_K8S-MAIN
          public: true
          pre_configured_consent_duration: 69y
          authorization_policy: two_factor
          redirect_uris:
            - http://localhost:8000
            # used if the port 8000 is already in use
            - http://localhost:18000
          scopes:
            - openid
            - profile
            - groups
            - email
          userinfo_signed_response_alg: none
          token_endpoint_auth_method: none
```

### Setup Kubernetes
Add these kube apiserver args.
In k3s, these can be altered by adding the following to `/etc/rancher/k3s/config.yaml`:
```yml
kube-apiserver-arg:
  - "oidc-issuer-url=https://auth.simulatan.me"
  - "oidc-client-id=k8s-main"
  - "oidc-username-claim=preferred_username"
  - "oidc-groups-claim=groups"
  # will prefix all groups coming from oidc with `oidc:`
  - "oidc-groups-prefix=oidc:"
```

Don't forget to create a role binding!
```yml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: oidc-kubernetes-admin-cluster-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: Group
  name: oidc:kubernetes-admin
```

### Setup kubectl
```bash
# `.config/kube/config` is the path to your kubeconfig file
kubectl config --kubeconfig .config/kube/config set-credentials oidc-user \
	--exec-api-version="client.authentication.k8s.io/v1beta1" \
	--exec-command="kubectl" \
	--exec-arg="oidc-login" \
	--exec-arg="get-token" \
	--exec-arg="--oidc-issuer-url=https://auth.simulatan.me" \
	--exec-arg="--oidc-client-id=k8s-main" \
	--exec-arg="--oidc-extra-scope=openid profile email groups"
```

Lastly, update your existing context to use this `oidc-user`.