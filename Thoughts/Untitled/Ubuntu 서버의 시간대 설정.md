---
title: Ubuntu 서버의 시간대 설정
created: 2025-05-23 09:38:24
author: andy
tags:
  - Ubuntu
---
## 1. 현재 시간대(Timezone) 확인

다음 명령어로 확인:

```bash
timedatectl
```

예시 출력:

```pgsql
Local time: Thu 2025-05-23 02:30:45 UTC
Time zone: UTC (UTC, +0000)
```

→ `Time zone: UTC` 라고 나오면 현지 기준으로 설정이 안 된 상태입니다.

---

## 2. 시간대 목록에서 원하는 지역 확인

```bash
timedatectl list-timezones | grep Seoul
```

출력 예시:

```
Asia/Seoul
```

---

## 3. 시간대를 Asia/Seoul (KST)로 변경

```bash
sudo timedatectl set-timezone Asia/Seoul
```

---

## 4. 적용 확인

```bash
timedatectl
```

예시:

```pgsql
Local time: Thu 2025-05-23 11:30:45 KST
Time zone: Asia/Seoul (KST, +0900)
```

___

## 5. 로그 시간 등도 자동으로 현지 시간 기준으로 표시됨

- `date`, `journalctl`, 메일 제목/본문의 시간 등
    
- `who`, `last`, `w` 등의 출력도 KST 기준으로 바뀜
