---
title: Ubuntu 서버로 SSH 접속 시 자동 메일 발송 설정
created: 2025-05-22 15:45:04
author: andy
tags:
  - Ubuntu
  - Security
---
## 1. 필요한 패키지 설치

```bash
sudo apt update
sudo apt install msmtp msmtp-mta mailutils
```
> `msmtp`: SMTP 클라이언트  
> `mailutils`: `mail` 명령어 제공

`msmtp` 설치 도중 'Enable AppArmor support?' 를 묻는 화면이 뜨면?

- AppArmor(애플리케이션 권한 제어 프레임워크)를 적용할지 여부를 묻는 것입니다

- `No` 권장

---

## 2. `msmtp` 설정

설정 파일 경로:

```bash
~/.msmtprc     ← 사용자 전용
또는
/etc/msmtprc   ← 시스템 전체 설정
```

예: Gmail SMTP 사용 예시 (`/etc/msmtprc` 기준)

```bash
sudo nano /etc/msmtprc
```

내용:

```ini
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        /var/log/msmtp.log

account        gmail
host           smtp.gmail.com
port           587
from           your_email@gmail.com
user           your_email@gmail.com
password       your_gmail_app_password

account default : gmail
```
>  **Gmail은 일반 비밀번호가 아닌 앱 비밀번호 필요**  
> → https://myaccount.google.com/apppasswords 에서 생성

권한 설정:

```bash
sudo chmod 600 /etc/msmtprc
sudo touch /var/log/msmtp.log
sudo chmod 666 /var/log/msmtp.log
```

---

## 3. 테스트 메일 보내보기

```bash
echo "테스트 메일입니다" | mail -s "SSH 알림 테스트" your_email@gmail.com
```

→ 메일이 수신되면 설정 성공
→ 수신이 안된다면 mail 앞에 `sudo` 붙여서 시도

---

## 4. SSH 로그인 시 메일 자동 발송

#### 방법 A. 사용자 로그인 시만 알림

```bash
sudo nano /home/username/.bashrc
```

맨 아래에 다음 추가:

```bash
echo "[$(date)] SSH 로그인 by $USER from $(who -m | awk '{print $NF}' | tr -d '()')" \
  | mail -s "[ALERT] SSH 로그인 감지 ($USER)" your_email@gmail.com
```
> `who -m`은 현재 로그인 IP를 가져옵니다.

---

#### 방법 B. 모든 사용자 로그인 시 알림 (PAM 전역 적용)

```bash
sudo nano /etc/pam.d/sshd
```

맨 아래에 추가:

```pam
session optional pam_exec.so /usr/local/bin/ssh-login-alert.sh
```

→ 알림 스크립트 생성:

```bash
sudo nano /usr/local/bin/ssh-login-alert.sh
```

```bash
#!/bin/bash

# 공통 정보
IP=$(who -m | awk '{print $NF}' | tr -d '()')
IP=${IP:-"unknown"}
TIME=$(date)
MAILTO="your_email@example.com"

case "$PAM_TYPE" in
    open_session)
        echo -e "[$TIME] SSH 로그인 감지\n사용자: $PAM_USER\nIP: $IP" \
            | mail -s "[ALERT] SSH 로그인: $PAM_USER@$IP" "$MAILTO"
        ;;
    close_session)
        echo -e "[$TIME] SSH 로그아웃 감지\n사용자: $PAM_USER\nIP: $IP" \
            | mail -s "[INFO] SSH 로그아웃: $PAM_USER@$IP" "$MAILTO"
        ;;
esac
```
> 변수 사용 시 `"..."` 안에 쓸 때는 `$MAILTO` 그대로 사용 가능
> 변수를 큰따옴표(`"`)로 감싸는 이유는 이메일 주소에 공백이 있을 때 안전하게 처리하기 위함입니다

여러 명에게 동시에 보내려면:

```bash
MAILTO="admin@example.com,security@example.com"
```
> 쉼표로 구분하면 여러 명에게 동시에 메일 발송됩니다 (메일 시스템이 이를 지원할 경우)

파일 권한 확인:

```bash
sudo chmod +x /usr/local/bin/ssh-login-alert.sh
```

___

## 5. 로그인 실패 시에도 알림 받기

기본적으로 **PAM에 등록된 `pam_exec.so`는 "세션이 성공적으로 시작될 때만" 실행되므로,**  
**로그인 실패 시에는 이메일이 발송되지 않습니다.**

```pam
session optional pam_exec.so /usr/local/bin/ssh-login-alert.sh
```

위 줄은 SSH 세션이 **성공적으로 시작되었을 때만** 실행됩니다.  
즉, 비밀번호 틀리거나 2FA 실패 등 **인증 실패 시점**에는 **이 스크립트가 실행되지 않습니다.**

---

#### 1. `fail2ban` 에서 특정 jail 이 IP를 차단 시 이메일 알림 설정

`fail2ban`은 로그 파일에서 로그인 실패를 감지하므로  
**SSH 비밀번호, OTP 실패, brute-force 시도까지도 감지**할 수 있습니다.

파일 열기:

```bash
sudo nano /etc/fail2ban/jail.local
```

예시: SSH 감시 설정에 메일 알림 추가

```ini
[sshd]
enabled = true
port    = ssh
filter  = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600

# 이메일 알림 action 지정
action = %(action_mwl)s

# 메일 관련 설정
destemail = your_email@example.com
sender = server@yourdomain.com
mta = mail
```
> `mta = mail` 은 `mail` 명령을 사용하도록 설정 → 내부적으로 `msmtp` 호출됨


#### 2. `action_mwl` 가 없다면 직접 만들기

파일 위치:

```bash
sudo nano /etc/fail2ban/action.d/action_mwl.conf
```

내용 예시:

```ini
[Definition]

# 이메일 제목
actionban = printf "Fail2Ban: <name> banned <ip> on <hostname>\n\nLog:\n<matched_logs>" \
  | mail -s "[Fail2Ban] <name>: banned <ip> on <hostname>" <dest>

actionunban = printf "Fail2Ban: <name> unbanned <ip> on <hostname>\n\n" \
  | mail -s "[Fail2Ban] <name>: unbanned <ip> on <hostname>" <dest>

# Optional
actionstart =
actionstop =
actioncheck =

[Init]
name = default
```
> 메일 본문에 `<matched_logs>`가 포함되며, 차단된 IP, jail 이름, 서버명 등을 포함한 제목으로 메일이 발송됩니다.

퍼미션 및 형식 점검:

```bash
sudo chmod 644 /etc/fail2ban/action.d/action_mwl.conf
sudo systemctl restart fail2ban
```

#### 2. 이메일 테스트용 `banaction` 수동 실행 확인

```bash
sudo fail2ban-client set sshd banip 1.2.3.4
```

→ `1.2.3.4`가 강제로 차단되며, 메일이 도착해야 함

#### 3. `fail2ban` 재시작

```bash
sudo systemctl restart fail2ban
```
