---
title: Docker 컨테이너 내부의 시간대를 호스트 시스템과 일치시키는 방법
created: 2025-05-23 09:45:38
author: andy
tags:
  - Ubuntu
  - Docker
---
## 1. `-v /etc/localtime:/etc/localtime:ro` 방식 (가장 일반적)

방법:

```bash
docker run -v /etc/localtime:/etc/localtime:ro ...
```

**설명:**

- 호스트의 `/etc/localtime` 파일을 컨테이너에 **읽기 전용으로 마운트**합니다.
    
- `localtime`은 시스템의 시간대 정보를 담고 있는 파일로, 이를 공유하면 컨테이너도 동일한 시간대를 사용하게 됩니다.
    

**장점:**

- 간단하고 정확하게 호스트 시간대와 동일하게 유지됩니다.
    
- 컨테이너 내 `date`, `timedatectl`, `tzdata`가 모두 호스트 기준으로 작동.

---

## 2. `TZ` 환경 변수 사용

방법:

```bash
docker run -e TZ=Asia/Seoul ...
```

또는 `docker-compose.yml` 작성시 다음 내용 추가:

```yaml
environment:
  - TZ=Asia/Seoul
```
> 단, 이 경우는 **컨테이너 내부에 `tzdata`가 있어야 반영됨**  
> Alpine 기반에는 기본적으로 없기 때문에 `apk add tzdata`를 별도로 실행하거나 Dockerfile에서 추가해줘야 합니다.

**설명:**

- 컨테이너 실행 시 환경 변수 `TZ`를 설정하면, 대부분의 리눅스 기반 컨테이너에서 해당 시간대를 사용합니다.
    
- `tzdata` 패키지가 설치되어 있어야 효과가 있음.
    

**장점:**

- 시스템과 다른 시간대를 설정하고 싶은 경우 유용함.
    

**예:**

```bash
docker run -e TZ=Asia/Seoul ubuntu date
```

---

## 3. Dockerfile에서 시간대 설정

Dockerfile을 사용하는 경우 다음과 같이 작성할 수 있습니다:

```Dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y tzdata && \
    ln -fs /usr/share/zoneinfo/Asia/Seoul /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata
```

**설명:**

- 이미지를 빌드할 때 시간대 설정을 고정하고자 할 때 사용.
    
- 주의: 이 방식은 이미지 고정이므로 **호스트와 자동 동기화되지는 않음**.

---

## 4. `/etc/timezone` 마운트 (보조 방법)

```bash
-v /etc/timezone:/etc/timezone:ro
```

- 일부 시스템은 `/etc/timezone` 파일을 참조해 시간대를 결정하므로, 이 파일도 함께 마운트하는 경우가 있음.
    
- 하지만 `/etc/localtime` 마운트만으로도 대부분의 경우 충분함.

---

## 요약 비교

|방법|동기화 여부|유연성|권장 상황|
|---|---|---|---|
|`/etc/localtime` 마운트|호스트와 자동 동기화|낮음|가장 일반적이고 정확|
|`TZ` 환경 변수|수동 설정|높음|개별 시간대 지정 필요 시|
|Dockerfile 설정|고정 설정|낮음|재사용용 이미지 제작 시|
|`/etc/timezone` 마운트|보조적|낮음|특정 배포판 호환성 문제 대응 시|

---

## 적용 확인

호스트 OS 의 시간대 설정 확인:

```bash
date
```

