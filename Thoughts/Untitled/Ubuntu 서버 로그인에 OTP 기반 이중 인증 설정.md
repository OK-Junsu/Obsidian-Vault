---
title: Ubuntu 서버 로그인에 OTP 기반 이중 인증 설정
created: 2025-05-22 13:24:25
author: andy
tags:
  - Ubuntu
  - Security
---
## 1. Google Authenticator PAM 모듈 설치

```bash
sudo apt update
sudo apt install libpam-google-authenticator
```

## 2. 사용자 계정에 OTP 등록

각 사용자가 로그인 시 OTP를 요구하게 하려면 **각 계정별로 설정**이 필요합니다.

```bash
google-authenticator
```

명령을 실행하면 다음과 같은 절차가 나타납니다:

1. QR 코드와 시크릿 키 제공 → 스마트폰 앱(Google Authenticator, Aegis 등)으로 등록
    
2. 백업 코드 제공
    
3. 보안 질문에 `y/n` 응답:
    
	- 시간 기반 사용 여부 (`time-based` → `y`)
	    
	- `.google_authenticator` 파일을 덮어쓸 것인지 (`y`)
	    
	- 여러 장비에서 재사용 허용 여부 (`n` 권장)
	    
	- 로그인 시 OTP 요구 여부 (`y`)

	- 시간 오차가 심할 경우, 최대 17개 코드까지(±4분 범위) 허용 여부 (`n` 권장)
	    
	- rate-limiting 허용 (`y`)

## 3. PAM 설정 파일 수정

PAM은 `/etc/pam.d/sshd` 또는 `/etc/pam.d/common-auth`를 사용해 인증 흐름을 제어합니다. 여기서는 SSH 로그인을 대상으로 설명합니다.

```bash
sudo nano /etc/pam.d/sshd
```

맨 아래에 다음 한 줄 추가:

```bash
auth required pam_google_authenticator.so
```

## 4. SSH 설정 파일 수정

```bash
sudo nano /etc/ssh/sshd_config
```

아래 항목을 다음과 같이 수정:

```bash
KbdInteractiveAuthentication yes
ChallengeResponseAuthentication yes
UsePAM yes
```

**OTP만 사용하는 것이 아니라 비밀번호 + OTP 조합(2FA)을 원할 경우**, 다음도 함께 확인:

```bash
PasswordAuthentication yes
```


## 5. SSH 구성 유효성 검사

다음 명령으로 구문 오류가 없는지 검사:

```bash
sudo sshd -t
```

→ 출력이 없으면 정상입니다.

## 6. SSH 서비스 재시작

```bash
sudo systemctl restart ssh
```

## 7. SSH 데몬 설정 확인

설정 적용 확인을 위해 다음 명령으로 SSH 데몬 설정을 확인할 수도 있습니다:

```bash
sshd -T | grep challenge
```

출력:

```nginx
challengeresponseauthentication yes
```

위와 같이 출력되면 성공적으로 적용된 것입니다. 하지만 몇몇 OpenSSH 버전에서는 `challenge` 키워드를 `sshd -T` 명령이 출력하지 않는 버그 또는 제한도 존재합니다.

## 8. SSH 접속 테스트

외부에서 SSH 시도 시 다음처럼 나와야 정상:

```css
user@server's password:
Verification code:
```

> 만약 OTP 입력 없이 비밀번호만 물어본다면 설정이 아직 적용되지 않은 것입니다.

## 9. 보안 강화를 위한 추가 권장 설정

#### `~/.google_authenticator` 권한 제한

```bash
chmod 600 ~/.google_authenticator
```

#### Fail2Ban 예외 설정 (자신의 IP 차단 방지)

```bash
sudo nano /etc/fail2ban/jail.local
```

```ini
[sshd]
ignoreip = 127.0.0.1 192.168.0.0/24 [자신의 공인IP]
```

→ 저장 후:

```bash
sudo systemctl restart fail2ban
```

## 10. 참고 및 보안 권장사항

| 항목               | 설명                                                |
| ---------------- | ------------------------------------------------- |
| **루트 계정 OTP 설정** | root 계정에도 `google-authenticator` 설정이 별도로 필요       |
| **Fail2Ban 연동**  | OTP 실패 로그를 추적하여 brute-force 차단 가능                 |
| **TOTP 앱 추천**    | Aegis(오픈소스), FreeOTP, Google Authenticator        |
| **백업코드 보관**      | `.google_authenticator` 파일과 백업코드는 안전하게 저장         |
| **다중 사용자 적용**    | 사용자마다 `google-authenticator` 실행 필요, 자동화는 스크립트로 가능 |
