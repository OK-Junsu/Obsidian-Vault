---
title: Ubuntu 서버 로그인에 SSH 공개키 인증 방식 사용
created: 2025-05-22 14:12:16
author: andy
tags:
  - Ubuntu
  - Security
---
SSH 로그인 시 **비밀번호 대신 SSH 공개키 인증 방식**을 사용하면,

- 보안성이 **크게 향상**되고
    
- OTP와도 **병행 가능**하며
    
- 자동화된 로그인 (스크립트, 백업 등)에서도 매우 유용합니다.

## 1. 클라이언트에서 SSH 키 생성 (없을 경우)

클라이언트 (로컬 PC)**에서 다음 명령 실행:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
> 추천: `ed25519`는 보안성과 속도 모두 우수합니다.
> 기존 `~/.ssh/id_ed25519`를 덮지 않으려면 다른 이름을 지정하세요.

키 생성 과정 중:

1. `Enter file...` 프롬프트에서 그냥 Enter 누르면 기본 경로에 생성됨:

	- 공개키: `~/.ssh/id_ed25519.pub`
	    
	- 개인키: `~/.ssh/id_ed25519`

2. `Enter passphrase (empty for no passphrase)` :

	- SSH 키를 생성할 때, **개인키에 추가적인 암호(패스프레이즈)**를 걸 수 있습니다.
	
	- **패스프레이즈를 입력하면:**  
	    SSH 키를 사용할 때마다 **비밀번호 입력을 요구**, 보안 강화  
	    (특히 개인키 유출 대비에 유리)
	    
	- **Enter 키를 눌러 비워두면:**  
	    로그인 자동화에는 편리하지만, **키 자체가 탈취되면 누구나 사용 가능**

## 2. 공개키를 서버로 복사

클라이언트에서 다음 명령 실행:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@ipaddress -p portnumber
```
> 비밀번호를 1회 입력해야 합니다.  
> 해당 키는 서버의 `~/.ssh/authorized_keys`에 등록됩니다.

**수동 복사도 가능**:

```bash
cat ~/.ssh/id_ed25519.pub | ssh username@ipaddress -p portnumber "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && chmod 700 ~/.ssh"
```

## 3. 서버의 SSH 설정 확인 (`/etc/ssh/sshd_config`)

```bash
sudo nano /etc/ssh/sshd_config
```

다음 항목이 명시적으로 있어야 합니다:

```conf
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

→ 이미 기본값으로 작동하는 경우도 있지만, 명시하면 더 명확함.

## 4. SSH 데몬 재시작

```bash
sudo systemctl restart ssh
```

## 5. 키로 로그인 테스트 (비밀번호 없이)

클라이언트에서:

```bash
ssh username@ipaddress -p portnumber
```

→ 자동으로 `~/.ssh/id_ed25519`를 사용해서 로그인 시도됨.  
→ 비밀번호 없이 바로 접속되면 성공!

## 선택적 설정: 비밀번호 로그인 비활성화

**로그인을 SSH 키 전용으로 제한하고 싶다면**:

```conf
PasswordAuthentication no
```

→ 이 설정은 **root나 다른 사용자가 SSH 키를 등록하지 않았다면 완전 접속 불가**가 되므로, **주의해서 적용**하세요.

```bash
sudo systemctl restart ssh
```

## OTP와 병행 사용도 가능

 SSH 키로 인증 + OTP를 추가로 요구할 수 있습니다.
 이 경우 `AuthenticationMethods` 설정 사용:

```bash
sudo nano /etc/ssh/sshd_config
```

```conf
AuthenticationMethods publickey,keyboard-interactive
```

→ SSH 키 + OTP를 모두 요구 (비밀번호는 안 씀)