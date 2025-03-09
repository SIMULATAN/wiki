---
title: MinIO
description: 
published: true
date: 2025-03-09T09:54:30.440Z
tags: 
editor: markdown
dateCreated: 2025-01-17T21:00:07.258Z
---

## Hosting on a single domain with Traefik
Using the same (sub)domain for both the API and the UI is not officially supported. The setup is quite scuffed and tricky. Here's a working `docker-compose.yml` that implements this using Traefik as the reverse proxy.

```yml
services:
  minio:
    image: docker.io/bitnami/minio:2025
    restart: unless-stopped
    hostname: minio
    expose:
      - 9000
      - 9001
    networks:
      - traefik
    volumes:
      - ./data:/bitnami/minio/data
    environment:
    	# TODO: set other variables required by minio
      MINIO_BROWSER_REDIRECT_URL: https://s3.example/minio
    labels:
      traefik.enable: true

      traefik.http.routers.minio.entrypoints: https
      traefik.http.routers.minio.tls.certresolver: pertls
      traefik.http.routers.minio.rule: Host(`s3.example.com`)
      traefik.http.routers.minio.service: minio
      traefik.http.services.minio.loadbalancer.server.port: 9000

      traefik.http.routers.minio-ui.entrypoints: https
      traefik.http.routers.minio-ui.tls.certresolver: pertls
      traefik.http.routers.minio-ui.rule: Host(`s3.example.com`) && PathPrefix(`/minio`)
      traefik.http.routers.minio-ui.service: minio-ui
      traefik.http.routers.minio-ui.middlewares: minio-ui-replacepath@docker
      traefik.http.services.minio-ui.loadbalancer.server.port: 9001
      traefik.http.middlewares.minio-ui-replacepath.stripprefix.prefixes: /minio


networks:
  traefik:
    external: true
```


## OIDC Config
Go to Identity -> OpenID and create an entry for Authelia.
![minio_oidc_configuration.png](/minio_oidc_configuration.png)