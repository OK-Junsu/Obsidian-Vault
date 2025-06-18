---
title: Ubuntu 서버의 계정 비밀번호 변경 방법
created: 2025-05-22 15:34:38
author: andy
tags:
  - Ubuntu
  - Security
---
## 1. 현재 로그인한 사용자 본인의 비밀번호 변경

```bash
passwd
```
> 프롬프트가 뜨며 현재 비밀번호와 새 비밀번호를 묻습니다.

## 2. 다른 사용자의 비밀번호 변경

→ **root 권한 필요** (또는 `sudo` 사용)

```bash
sudo passwd username
```
> 바로 새 비밀번호만 입력하면 됩니다 (기존 비밀번호 확인 없이 변경 가능).

## 3. root 계정 비밀번호 변경

#### 방법 A: 일반 사용자로 로그인한 상태에서

```bash
sudo passwd root
```

→ 새 root 비밀번호 입력

#### 방법 B: root 계정으로 직접 로그인한 상태에서

```bash
sudo -u root -i
passwd
```
 > 이 경우 root 자신의 비밀번호를 바꾸게 됩니다.
 
