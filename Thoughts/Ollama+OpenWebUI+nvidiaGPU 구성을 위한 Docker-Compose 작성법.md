---
title: Ollama+OpenWebUI+nvidiaGPU 구성을 위한 Docker-Compose 작성법
created: 2025-06-13 09:54:21
author: andy
tags:
  - Docker
---
```yml
version: "3.3"
services:
  ollama:
    container_name: ollama
    image: ollama/ollama:${OLLAMA_DOCKER_TAG-latest}
    ports:
      - 11434:11434
    pull_policy: always
    tty: true
    restart: unless-stopped
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /docker/ai/ollama:/root/.ollama
    environment:
      - OLLAMA_KEEP_ALIVE=24h
    #      - ENABLE_IMAGE_GENERATION=True
    #      - COMFYUI_BASE_URL=http://stable-diffusion-webui:7860
    deploy:
      resources:
        reservations:
          devices:
            - driver: ${OLLAMA_GPU_DRIVER-nvidia}
              count: ${OLLAMA_GPU_COUNT-1}
              capabilities:
                - gpu
    devices:
      - /dev/nvidia-uvm
      - /dev/nvidia-uvm-tools
      - /dev/nvidia-modeset
      - /dev/nvidiactl
      - /dev/nvidia0
  open-webui:
    container_name: open-webui
    image: ghcr.io/open-webui/open-webui:${WEBUI_DOCKER_TAG-main}
    ports:
      - 11500:8080
    environment:
      - WEBUI_URL=https://ai.misolves.com
      - OLLAMA_BASE_URL=http://ollama:11434
      - ENABLE_OAUTH_SIGNUP=true
      - ENABLE_SIGNUP=false
      - ENABLE_LOGIN_FORM=false
      - OAUTH_PROVIDER_NAME=MSE Auth
      - OAUTH_CLIENT_ID=${OAUTH_CLIENT_ID}
      - OAUTH_CLIENT_SECRET=${OAUTH_CLIENT_SECRET}
      - OPENID_PROVIDER_URL=${OAUTH_URL}
      - OAUTH_SCOPES=openid email profile picture
      - DEFAULT_USER_ROLE=user
    depends_on:
      - ollama
    extra_hosts:
      - host.docker.internal:host-gateway
    restart: unless-stopped
    volumes:
      - /docker/ai/open-webui:/app/backend/data
#  searxng:
#    image: searxng/searxng:latest
#    container_name: searxng
#    ports:
#      - 11501:8080
#    volumes:
#      - /etc/localtime:/etc/localtime:ro
#      - /etc/timezone:/etc/timezone:ro
#      - /docker/ai/searxng:/etc/searxng
#    depends_on:
#      - ollama
#      - open-webui
#    restart: unless-stopped
networks: {}
x-dockge:
  urls:
    - https://ai.misolves.com
    - https://github.com/ollama/ollama
    - https://github.com/open-webui/open-webui
```