최초 작성: 2025-05-12 16:20:00 +09:00
___
## 1. 핵심 요약

VirtualBox 설치 및 네트워크 설정 변경이 `Workstation` 서비스 오류 및 시스템 이미지 생성 실패 원인과 **직접적으로 관련**이 있을 수 있습니다.

VirtualBox 설치 시 생성되는 **가상 네트워크 어댑터(VirtualBox Host-Only Network 등)** 또는 **브리지 설정 변경**이 Windows의 **네트워크 관련 시스템 서비스(LanmanWorkstation 포함)** 구성에 영향을 주었을 수 있습니다.

그 결과 **Workstation 서비스가 오작동**하거나 **종속 서비스 구성이 손상**되어 `0x807800A1`, `0x800423F4` 오류가 발생할 수 있습니다.

## 2. VirtualBox가 영향을 줄 수 있는 주요 항목

### 1. Workstation 서비스는 “네트워크 공유” 관련 서비스

- Workstation(LanmanWorkstation)은 **SMB 프로토콜 기반 파일 공유**, **원격 파일 시스템 접근**, **VSS 관련 ASR Writer**에 필요합니다.
    
- 이 서비스는 정상 작동을 위해 다음 요소에 의존:
    
    - 네트워크 어댑터 드라이버
    - `nsi` 서비스 (Network Store Interface)
    - TCP/IP Protocol Driver
    - NetBIOS over TCP/IP

### 2. VirtualBox 설치 시 다음 작업이 자동으로 수행됨

| VirtualBox 설치 시 작업   | 설명                                          |
| -------------------- | ------------------------------------------- |
| 가상 어댑터 생성            | VirtualBox Host-Only Adapter, NAT Network 등 |
| Bridged 설정 시 NIC 재구성 | 물리 네트워크 어댑터 구성 변경                           |
| Windows 드라이버 스택 삽입   | 필터 드라이버 삽입으로 인해 시스템 서비스 오작동 가능              |

## 3. 어떤 식으로 문제가 생기는가?

- **잘못된 NIC 바인딩** 또는 **필터 드라이버 충돌**로 인해,  
    `LanmanWorkstation`이 의존하는 **네트워크 서비스(nsi, TCP/IP 등)**에 접근 실패
- 또는, VirtualBox 설치 시 어댑터 변경으로 **서비스 등록 정보가 꼬이거나 삭제됨**
- 이로 인해 **Error 1075 / Error 2** 등 Workstation 서비스 오류 발생 → 백업 실패

## 4. 해결 방법

### 1. 네트워크 어댑터 재설정 및 불필요한 VirtualBox 어댑터 제거

1. `제어판 → 네트워크 및 공유 센터 → 어댑터 설정 변경`
    
2. 다음 항목 중 불필요한 항목 제거:
    
    - VirtualBox Host-Only Network
    - VirtualBox 브리지 어댑터
        
3. 실사용 LAN 어댑터에 아래 항목이 체크되어 있는지 확인:
    
    - **Client for Microsoft Networks**
    - **File and Printer Sharing**
    - **Internet Protocol Version 4 (TCP/IPv4)**

### 2. NIC 재설치 및 Winsock 초기화

```cmd
# Winsock 초기화 (네트워크 스택 복구)
netsh winsock reset

# TCP/IP 설정 초기화
netsh int ip reset
```
→ 재부팅 필수


### 3. Workstation 서비스 종속성 복구

```regedit
# 경로:
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation

# DependOnService 값 확인 또는 복원:
NSI
```

### 4. [선택] VirtualBox 제거 후 재설치 (서비스 충돌 해결용)

- `VirtualBox`를 제거하고 **남은 드라이버/가상 어댑터를 정리**
- 이후 `DISM`, `sfc` 수행 후, 백업 재시도

___

## 5. 결론

| 항목                 | 판단                                   |
| ------------------ | ------------------------------------ |
| VirtualBox와 오류 연관성 | 높음 (가상 어댑터 및 드라이버 구성 변경 때문)          |
| 주요 영향 요소           | 네트워크 스택, NSI 서비스, Workstation 의존성    |
| 복구 조치              | 가상 어댑터 정리, Winsock 초기화, 서비스 레지스트리 복원 |
| 최종 해결책             | In-place upgrade 설치 (전체 서비스 구조 복원)   |
