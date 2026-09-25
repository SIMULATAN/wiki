---
title: Talos
description: 
published: true
date: 2026-09-25T18:33:28.144Z
tags: 
editor: markdown
dateCreated: 2026-09-25T18:33:28.144Z
---

# Generating a new Client Configuration from Machine Secrets

```bash
# read machine secrets from cluster initialization
jq -r '                                                                         
  .resources[]
  | select(
      .mode == "managed" and
      .type == "talos_machine_secrets" and
      .name == "this"
    )
  | .instances[0].attributes.machine_secrets
' terraform.tfstate > secrets.yaml

# extract relevant data (cert may be stored under `.certs.os.crt` too)
yq -r '.certs.os.cert' secrets.yaml | base64 -d > ca.crt
yq -r '.certs.os.key' secrets.yaml | base64 -d > ca.key

talosctl gen key --name recovery-admin
talosctl gen csr --key recovery-admin.key --ip 127.0.0.1
talosctl gen crt --ca ca --csr recovery-admin.csr --name recovery-admin --hours 6969

# copy your old config to talosconfig_recovery

CA=$(base64 -w0 ca.crt)
CRT=$(base64 -w0 recovery-admin.crt)
KEY=$(base64 -w0 recovery-admin.key)
yq -i \                                                                   
  --arg ca "$CA" \
  --arg crt "$CRT" \
  --arg key "$KEY" \
  '.contexts[].ca = $ca
   | .contexts[].crt = $crt
   | .contexts[].key = $key' \
  ~/.config/talosctl/talosconfig_recovery

talosctl --talosconfig=~/.config/talosctl/talosconfig_recovery health
```