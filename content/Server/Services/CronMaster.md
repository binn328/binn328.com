---
title: CronMaster
description: CronMaster, CronJob 사용을 간편하게 해주는 앱
date: 2026-09-26
tags:
  - "#app/cronmaster"
aliases:
draft: false
permalink:
---
### 소개
![[Pasted image 20260926204330.png]]
[CronMaster](https://github.com/fccview/cronmaster)는 cronjob에 사용되는 bash 스크립트를 쉽게 관리하고 사용할 수 있도록 해주는 앱이다. 

### docker-compose
```yaml
services:
  cronmaster:
    image: ghcr.io/fccview/cronmaster:latest
    container_name: cronmaster
    user: root
    ports:
      - 40123:3000
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_CLOCK_UPDATE_INTERVAL=30000
      - AUTH_PASSWORD=${AUTH_PASSWORD}
      - HOST_CRONTAB_USER=root
      - MAX_LOG_AGE_DAYS=90 # Days to keep logs (default: 30)
      - MAX_LOGS_PER_JOB=120 # Maximum logs per job (default: 50)
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./scripts:/app/scripts
      - ./data:/app/data
      - ./snippets:/app/snippets
    pid: host
    privileged: true
    restart: always
    init: true
```

### 사용방법
사용 방법은 간단하다. `New Task` 버튼을 눌러 새로운 cronjob을 생성하고 테스트하면 끝이다.

![[Pasted image 20260926204346.png]]
현재 나는 블로그를 3시간마다 업데이트하는데 사용하고 있다. `Cloudflare Pages`는 매달 500번 빌드 한도를 제공하는데, 30일 x 하루 8번으로 한 달에 약 240번의 빌드를 수행하도록 구성을 해두었다. 남은 한도는 빠르게 무언가를 올릴 때 사용할 수 있도록 여유 분으로 남겨두었다.

