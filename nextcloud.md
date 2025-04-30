---
title: Nextcloud
description: 
published: true
date: 2025-04-30T08:25:19.831Z
tags: 
editor: markdown
dateCreated: 2024-04-13T16:35:37.456Z
---

# Nextcloud
Tips & Tricks related to Nextcloud

## Use `occ` in Docker / Kubernetes
`su -s /bin/sh www-data -c "php occ --version"`

## OIDC Setup
1. Create a new Authelia OIDC client
	 see [Authelia integration docs](https://www.authelia.com/integration/openid-connect/nextcloud/) and the [Authelia wiki page](/en/authelia)
2. Install the NextCloud App "OpenID Connect user backend". Notice that it's in the "Social" section of the App Store.
3. Configure OIDC
![nextcloud oidc configuration](/nextcloud_oidc_configuration.png)
