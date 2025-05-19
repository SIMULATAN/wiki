---
title: Authelia
description: Notes on Authelia
published: true
date: 2025-05-19T08:56:38.254Z
tags: 
editor: markdown
dateCreated: 2024-04-13T16:10:57.971Z
---

# Authelia
## Create a new Client in my K8s Cluster
1. Create secret: `docker run --rm authelia/authelia:latest authelia crypto hash generate pbkdf2 --variant sha512 --random --random.length 72 --random.charset rfc3986`
2. Add variables for secret in a new file
```
_CLIENT_SECRET_MYSERVICE=<random password>
CLIENT_SECRET_MYSERVICE=<digest>
```
Make sure to remember the random password as this is the one you have to give to the client. As such, the plain text password is also stored in the secret. Do note that this is kinda insecure, lol

3. merge new secrets into existing one:
```
kubectl create secret generic tempxxx \
	--dry-run=client \
  --from-env-file authelia-oidc-secrets.local.yml \
  -o yaml | kubeseal --format yaml --merge-into authelia-oidc-secrets.yml
```

4. order `authelia-oidc-secrets.yml` correctly & remove bloat from yaml
5. add client to `authelia-oidc-clients.yml` - see existing clients for reference