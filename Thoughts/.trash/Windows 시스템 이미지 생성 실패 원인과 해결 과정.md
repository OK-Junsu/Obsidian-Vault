최초 작성: 2025-05-12 11:29:00 +09:00
___
## 에러 유발 과정

![[Pasted image 20250512095139.png]]
> Backup and Restore (Windows 7)

![[Pasted image 20250512100607.png]]
> Create a system image

아래 에러 코드로 오류 발생

- 0x807800A1
- 0x800423F4

___

## 에러 코드 분석

### 1. `0x807800A1`

- **뜻:**  
	시스템 이미지 백업을 생성할 수 없음.  
    특히, **백업 대상 드라이브 또는 포함된 파티션 중 하나에 VSS 스냅샷 생성 실패** 시 발생함.
    
- **주된 원인:**
    
    - 복수 개의 **활성 파티션** 또는 시스템 예약 파티션이 있음
    - 백업하려는 디스크에 **충돌나는 복구 파티션**, **EFI 파티션**, 또는 **숨겨진 파티션**이 포함됨

### 2. `0x800423F4`

- **뜻:**  
    VSS가 **스냅샷을 생성하지 못함** (VSS_E_WRITERERROR_TIMEOUT 또는 WRITERERROR_NONRETRYABLE)
    
- **주된 원인:**
    
    - 특정 VSS Writer가 정상적으로 동작하지 않음 (예: `System Writer`, `Shadow Copy Optimization Writer`)
    - 오래된 백업 스케줄 잔재, 디스크 상태 문제, 시스템 드라이브 용량 부족

___

## 해결 과정 1

### 1단계: **VSS 서비스 확인 및 재시작**

```cmd
# 관리자 권한 CMD에서 실행
net stop vss
net start vss
```

___

### 2단계: **VSS Writer 상태 확인**

```cmd
vssadmin list writers
```

- 출력 결과 중 `State: [1] Stable` 인지 확인
- `Error`, `Retryable`, `Waiting for completion` 등이 있다면 문제 있는 Writer


**Write 문제 발생하여 아래 내용으로 조치**:

- 서비스 재시작 (`COM+ Event System`, `Volume Shadow Copy`, `Microsoft Software Shadow Copy Provider`)
- 심하면 `DISM` 및 `SFC`로 시스템 복구
	
	```cmd
	DISM /Online /Cleanup-Image /RestoreHealth
	sfc /scannow
	```

---

### 3단계: **백업 대상에서 복구 파티션 제외 (권장)**

- 일부 제조사 시스템에서는 **복구 파티션** 또는 숨겨진 파티션 때문에 오류 발생
- `diskmgmt.msc` 또는 `diskpart`로 파티션 확인

**권장 조치**:

- 이미지 백업 시, 시스템 드라이브(C:)와 EFI/시스템 예약 파티션만 선택
- 복구 파티션은 제외하는 방식으로 진행:  
	- 제어판 > 백업 및 복원 > 시스템 이미지 만들기 > 수동 선택 시 조정 가능

___

### 3-2단계: **복구 파티션 제외가 안될 때**

#### 제외할 수 없는 이유

- 시스템이 자동으로 포함시키는 파티션 목록:

	1. **EFI System Partition (ESP)** – 부팅 정보가 저장된 파티션
	2. **System Reserved Partition** – BitLocker, 부팅 구성 관련
	3. **Windows Recovery Environment (WinRE) 파티션** – 복구 기능용
	4. **숨겨진 제조사 복구 파티션** – OEM 복구용, 오류 유발 가능

이런 파티션은 시스템 이미지 백업 시, **필수로 포함되도록 설정**되어 있으며,  
제외 체크박스를 비활성화해서 **사용자가 해제하지 못하게 막아두었습니다**.

#### 해결 방법

백업 오류 유발 파티션 식별 및 숨기기

1. **디스크 파티션 구조 확인**
	```cmd
	diskpart
	list disk
	select disk 0       ← 보통 OS 설치된 디스크
	list partition
	```

2. 파티션 중 **Recovery**, **OEM**, 또는 용도 불명 파티션 확인  
    특히 크기가 500MB~1GB 정도인 것이 문제 되는 경우가 많습니다.
3. **파티션 ID를 변경하여 시스템 인식에서 제외**
	예시 - `diskpart`로 파티션 ID 변경 (시스템에서 무시되게 하기):

	```cmd
	# 관리자 CMD에서
	diskpart
	select disk 0
	list partition
	select partition <문제 파티션 번호>
	set id=DE94BBA4-06D1-4D40-A16A-BFD50179D6AC override
	```
> 이 GUID는 일반 데이터 파티션으로 설정해주는 값입니다. VSS에서 무시되게 만듭니다.

___

### 4단계: **백업 위치 및 디스크 확인**

- 외장하드 또는 NAS 등에 백업할 때 해당 드라이브에 쓰기 권한 있고, NTFS 포맷인지 확인
- 백업 대상 드라이브에 **시스템 예약 파티션 포함 금지**
- 백업할 드라이브에 **오류 검사 (chkdsk)** 수행

```cmd
chkdsk C: /f /r
```

___

### 5단계: **VSS 스냅샷 삭제 및 초기화**

```cmd
vssadmin delete shadows /all
```
> 모든 이전 스냅샷 삭제 → 디스크 여유 공간 확보 및 충돌 방지 목적

___

## 해결 과정 2

### 1. 이벤트 로그 기반 정밀 분석

1. `이벤트 뷰어 (eventvwr.msc)` 실행
2. 다음 경로로 이동: `Windows 로그 → 응용 프로그램`
3. `Source`가 `VSS`, `Backup`, 또는 `VolSnap` 인 이벤트 중에서 **오류(빨간색 X)** 찾아 확인
> **이벤트 ID 8193, 12289, 13, 22** 등은 직접적인 힌트를 줍니다.

___

### 2. 확인 된 이벤트 로그

```
521  
The backup operation that started at '‎2025‎-‎05‎-‎11T14:10:38.350000000Z' has failed because the Volume Shadow Copy Service operation to create a shadow copy of the volumes being backed up failed with following error code '0x807800A1'. Please review the event details for a solution, and then rerun the backup operation once the issue is resolved.

16387  
Writer ASR Writer experienced some error during snapshot creation. More info: The Workstation service has not been started. (0x8007085A).

8193  
Volume Shadow Copy Service error: Unexpected error calling routine Check OnIdentifyError. hr = 0x8007085a, The Workstation service has not been started.  
.

Operation:  
PrepareForBackup event

Context:  
Execution Context: ASR Writer  
Execution Context: Writer  
Writer Class Id: {be000cbe-11fe-4426-9c58-531aa6355fc4}  
Writer Name: ASR Writer  
Writer Instance ID: {1b2b58b4-cfc9-4557-9e23-5063f4741021}

Error-specific details:  
ASR Writer: The Workstation service has not been started. (0x8007085A)
```

#### 1. 로그 분석 결과

- **문제 원인:** `Workstation` 서비스가 꺼져 있어서 `ASR Writer`가 백업 중 VSS 작업을 완료하지 못한 것입니다.

- **해결 방법:** `Workstation` 서비스를 시작시키고, 다시 백업을 실행하면 됩니다.

___

#### 2. 로그 상세 분석

이벤트 ID: **8193 / 16387 / 521**

- 모두 **ASR Writer (Automated System Recovery Writer)** 관련 오류이며,
- `0x8007085A` → **"The Workstation service has not been started"** 라는 의미입니다.
    
ASR Writer란?

- Windows 시스템 이미지 백업 시 **전체 시스템 상태를 기록**하는 Writer입니다.
- 해당 Writer는 `Workstation` 서비스에 의존함 (특히 파일 공유 등)

___

### 3. 해결 방법 1

#### 1단계: `Workstation` 서비스 수동으로 시작

1. **Win + R → `services.msc` 입력 → 엔터**
2. **목록에서 "Workstation" 서비스 검색**
3. 더블 클릭 → **시작 유형을 '자동'으로 변경**
4. '시작' 버튼 클릭 후 적용
    

또는 명령어로 처리할 수도 있습니다:

	```cmd
	sc config LanmanWorkstation start= auto
	net start LanmanWorkstation
	```

___
#### 2단계: 다시 시스템 이미지 백업 시도

- `제어판 > 백업 및 복원 (Windows 7) > 시스템 이미지 만들기` 로 재시도
- 정상적으로 백업이 시작될 것입니다

___

#### 실패시

1. 다른 오류 발생

	Workstation 서비스 시작시 다음 에러메세지 나오며 실행 실패

	```plaintext
	windows could not start the workstation service on local computer error 1075: the dependency service does not exist or has been marked for deletion
	```

 2. 오류 코드 `1075` 의 원인 분석

	`Workstation` 서비스가 시작되려면 **의존하는 다른 서비스(종속 서비스)**가 먼저 정상이어야 하는데, 해당 종속 서비스가 삭제되었거나 비정상 상태라는 뜻입니다.

	**Workstation 서비스의 기본 종속 서비스**:
	
	- **Network Store Interface Service (`nsi`)**
	- **SMB-related 서비스 (예: Bowser, MRxSmb, etc.)**
	- 내부적으로는 **TCP/IP Protocol Driver**, **NetBIOS over TCP/IP**, 등도 연관

___

### 4. 해결 방법 2

#### 1단계: 현재 종속 서비스 확인

- 명령 프롬프트에서 다음 입력:

	```cmd
	sc qc lanmanworkstation
	```

- 출력 예시:

	```yaml
	[SC] QueryServiceConfig 성공
	
	SERVICE_NAME: lanmanworkstation
			...
			DEPENDENCIES       : Bowser
								 MRxSmb
								 NSI
	```
> 이 중 **존재하지 않거나 손상된 서비스가 있는지 확인**

- 실제 출력 결과: 아무런 출력도 없음

#### 2단계: 종속성 설정을 기본값으로 복원

- 정상적인 시스템에서의 기본 종속 서비스는: 
	
	```plaintext
	NSI
	```

- **레지스트리 수정을 통해 복구**할 수 있습니다.
- 방법: 레지스트리에서 종속성 재설정

	1. **`regedit` 실행** (`Win + R` → `regedit`)
	2. 이동:

		```sql
		HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation
		```

	3. 오른쪽에서 `DependOnService` 값을 더블 클릭
	4. 다음처럼 수정:

		```objectivec
		NSI
		```

	5. 저장 후 재부팅

#### 3단계: Workstation 서비스 재시작

- 이제 다시 다음 명령어 입력:

	```cmd
	net start LanmanWorkstation
	```
> 정상적으로 시작된다면 문제 해결입니다.

___

### 5. 해결 방법 3

#### 1. 해결 방법 2의 3단계에서 실패 후 아래 메세지 출력

```plaintext
The Workstation service is starting. The Workstation service could not be started. A system error has occurred. System error 2 has occurred. The system cannot find the file specified.
```

#### 2. 원인:

`LanmanWorkstation` 서비스의 실행 파일 또는 핵심 드라이버가 손상되었거나, 경로가 잘못되었거나, 시스템에서 완전히 누락됨.

이 상황에서는 **레지스트리 수동 복원** 또는 **Windows 자체 복구 설치 (In-place upgrade)** 외에는 서비스 복구가 불가능합니다.  
(서비스는 단순한 실행 파일이 아닌, 시스템 드라이버와 레지스트리 구성이 함께 작동해야 하기 때문입니다.)

#### 3. 방법 A:

최근 VirtualBox 를 설치하거나 VirtualBox 를 통해 네트워크 설정을 수정한 이후 문제가 발생한 경우에는 아래 링크의 문서를 참고 할 것

[[Workstation 서비스의 관련 요소 손상과 VirtualBox 의 연관성]]

#### 4. 방법 B: LanmanWorkstation 서비스 수동 복원 (레지스트리)

이 방법은 **정상적인 Windows 시스템**에서 `LanmanWorkstation` 서비스의 레지스트리 설정을 내보내서 문제 있는 시스템에 다시 **수동 병합(import)**하는 방식입니다.

- 경로:

	```sql
	HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation
	```
- `LanmanWorkstation` 키 전체를 `.reg` 파일로 내보내기

#### 5. 방법 C: In-place Upgrade 설치 (권장)

**가장 안전하고 확실한 방법**입니다. Windows 설치 원본(ISO)을 사용해 시스템을 초기화 없이 복구합니다.

1. [Microsoft 공식 Windows ISO 다운로드](https://www.microsoft.com/software-download/)
    
    - Windows 10 또는 11에 맞게 선택
        
2. 다운로드된 ISO 파일 **더블 클릭** → 가상 드라이브로 마운트됨
    
3. 루트에 있는 `setup.exe` 실행
    
4. "개인 파일 및 앱 유지" 옵션 선택
    
5. 설치 진행 → 재부팅 후 모든 서비스 복구됨

> 이 방식은 `LanmanWorkstation`, VSS 구성요소, 레지스트리 등 모두 복구됨

___
## 해결 과정 3

### 1. `0x80780119` 에러 의미와 원인

- 이제는 아래 에러 메세지와 함께 오류 코드가 `0x80780119`로 바뀌었다. 
	
	```plaintext
	there is not enough disk space to create the volume shadow copy on the storage location. make sure that, for all volumes to be backup up, the minimum required disk space for shadow copy craetion is available. this applies to both the backup storage distination and volumes included in the backup. minimum requirement: for volumes less than 500 megabytes, the minimum is 50 megabytes of free space. for volumes more then 500 megabytes ...
	```

- **의미**:
	Windows가 백업 도중 **VSS(볼륨 섀도우 복사본)**을 만들 수 없을 정도로 **디스크 공간이 부족하거나 파티션 구성이 잘못됨**

- **주요 원인 요약**:

|     | 원인                           | 설명                       |
| --- | ---------------------------- | ------------------------ |
| 1   | 시스템 예약 파티션에 여유 공간 부족         | 500MB 미만 파티션은 최소 50MB 필요 |
| 2   | 백업 저장 위치의 여유 공간 부족           | 외장하드, 다른 드라이브 등          |
| 3   | EFI 파티션이나 복구 파티션에 VSS 공간 부족  | 일반 사용자에겐 숨겨져 있음          |
| 4   | 볼륨에 할당된 VSS shadowstorage 부족 | 예약 공간이 작게 잡혀 있거나 없음      |

___

### 2. `0x80780119` 에러 해결 방법 1

#### 1단계: 모든 드라이브 여유 공간 확인

- 특히 **시스템 예약 파티션 (통상 100MB ~ 500MB)** 의 **남은 공간이 50MB 이상**인지 확인 필요
    
	확인 방법:

	```cmd
	mountvol
	```
	
	→ 시스템 예약 파티션이 있는 드라이브에 문자 할당되어 있지 않으면, 임시로 할당:
	
	```cmd
	mountvol R: /S
	```
	
	→ 그 후 `diskmgmt.msc` 또는 `File Explorer`로 R: 확인 → 속성 → 공간 확인

___

#### 2단계 ### Shadow Storage 공간 재할당

```cmd
vssadmin list shadowstorage
```

→ 백업 대상 드라이브(C: 등)에 `Allocated` 값이 작게 잡혀 있으면 다음 실행:

```cmd
vssadmin resize shadowstorage /for=C: /on=C: /maxsize=10%
```
> 위 명령은 C: 드라이브에 VSS 공간을 전체 용량의 10%로 설정

___

#### 실패시

Windows 10/11에서 시스템 예약 파티션의 여유 공간이 부족할 때 `0x80780119` 오류가 거의 항상 발생하므로 어떤 파티션이 부족한지 구체적인 분석이 필요.

##### 1. 파티션 분석

- 다음 명령어로 각 디스크의 파티션 구성을 확인해 주세요:

	```cmd
	diskpart
	list disk
	
	select disk 0
	list partition
	
	select disk 1
	list partition
	
	select disk 2
	list partition
	
	...
	```

- 이 정보로:

	- **시스템 예약 파티션**이 존재하는지
	- EFI, 복구 파티션의 크기와 위치
	- C: 드라이브가 어느 디스크에 속해 있는지
    
	를 알 수 있습니다.

- 출력 결과:

	```cmd
	list disk
	
	Disk ### Status Size Free Dyn Gpt
	
	-------- ------------- ------- ------- --- ---
	
	Disk 0 Online 7452 GB 1024 KB *  
	Disk 1 Online 1863 GB 1024 KB *  
	Disk 2 Online 1863 GB 1024 KB *
	
	select disk 1
	
	Partition ### Type Size Offset
	
	------------- ---------------- ------- -------
	
	Partition 1 System 100 MB 1024 KB  
	Partition 2 Reserved 16 MB 101 MB  
	Partition 3 Primary 1862 GB 117 MB  
	Partition 4 Recovery 644 MB 1862 GB
	```

___

##### 2. 파티션 분석 결과

**파티션 구조 분석 (디스크 1 기준)**

| 파티션 번호 | 유형       | 크기      | 용도 예측                   |
| ------ | -------- | ------- | ----------------------- |
| 1      | System   | 100 MB  | **시스템 예약 파티션** (EFI 아님) |
| 2      | Reserved | 16 MB   | Microsoft 예약 파티션 (MSR)  |
| 3      | Primary  | 1862 GB | **C: 드라이브** (운영체제 파티션)  |
| 4      | Recovery | 644 MB  | Windows 복구 환경 (WinRE 등) |

문제 원인:  

파티션 1 (System 파티션)이 100MB로 너무 작아, VSS가 스냅샷을 만들 공간이 부족합니다.

- Windows는 시스템 이미지 백업 시 **부팅 관련 파티션까지 포함**합니다.
    
- 100MB 파티션은 Windows 10 이상에서 점차 비표준이 되었고,  
    VSS가 요구하는 **최소 여유 공간 50MB**를 충족하지 못하는 경우 오류가 발생합니다.
    
- 게다가 일부 시스템에선 VSS 관련 로그 파일이나 템포러리 데이터도 이 파티션에 기록되기 때문에  여유 공간이 40~50MB 미만이면 백업이 실패합니다.

___

##### 3. 해결 방법

1. ~~방법 1: ### 시스템 예약 파티션 확장 (권장)~~

	- **MiniTool Partition Wizard Free**, **EaseUS Partition Master Free** 등 설치

	- **파티션 3 (C 드라이브)**의 앞쪽에서 200~300MB 공간 확보

	- 확보된 공간을 **파티션 1에 붙여서 260MB 이상으로 확장**

	- 적용 후 재부팅

> 시스템 파티션이므로 **작업 전 전체 백업 권장**
> 파티션 확장은 **Windows 디스크 관리(diskmgmt.msc)**에서는 불가능 (GUI 제한)

___

2. 방법 2: 복구 파티션을 임시 비활성화 → 백업 완료 후 복원

	```cmd
	# WinRE 비활성화
	reagentc /disable
	```

	이후 백업 시도 → 성공 → 백업 완료 후 복원:

	```cmd
	# WinRE 다시 활성화
	reagentc /enable
	```

___

#### 참고

**VSS 저장소가 C 드라이브에 설정되어 있다면**, 단순히 **C 드라이브의 Shadow Copy 공간 부족**이 원인일 수 있고,  **"시스템 예약 파티션 확장"은 무관한 제안처럼 보일 수 있습니다.**

하지만 실제로는 **두 요소 모두 `0x80780119` 오류에 관여할 수 있으며**, 각각 다음과 같은 역할을 합니다:

___
##### 1. 왜 시스템 예약 파티션이 관계있을 수도 있는가?

1. Windows 시스템 이미지 백업은 단순히 C 드라이브만 백업하지 않음

	1. 백업 대상에는 **C 드라이브 외에도 부팅 관련 파티션 (예: 시스템 예약 파티션)**이 자동 포함됩니다.
	2. 시스템 이미지 백업은 **OS 복구 가능 상태로 만들기 위해** 부트 로더가 위치한 파티션까지 백업합니다.
	3. - 이 과정에서 **그 파티션에도 VSS 스냅샷을 시도**합니다.

2. 시스템 예약 파티션에도 **약간의 여유 공간(VSS 메타데이터용)이 필요**

	1. 100MB짜리 시스템 예약 파티션은 Windows 10/11 기준으로 **거의 꽉 차 있는 경우가 많음**
	2. VSS는 메타데이터/로그 파일을 **스냅샷 생성 순간 잠시 기록**하려고 시도 → 공간 부족 시 실패
	3. 이 경우에도 `0x80780119` 발생 가능

> (※ 백업은 VSS 오류가 하나라도 발생하면 전체 실패 처리함)

___

##### 2. 정리

- **C 드라이브의 VSS 저장소 부족** → 직접적 원인  
- **시스템 예약 파티션 공간 부족** → 간접적 또는 병렬 원인 (스냅샷 생성 실패)

___

##### 3. 검증 방법

1. C 드라이브의 Shadow Copy 공간 충분한지 확인

	```cmd
	vssadmin list shadowstorage /for=C:
	```

	→ `Used`, `Allocated`, `Maximum` 확인  
	→ 충분하지 않다면:

	```cmd
	vssadmin resize shadowstorage /for=C: /on=C: /maxsize=10%
	```

2. 시스템 예약 파티션에 VSS 설정 여부 확인
	1. `mountvol`로 시스템 예약 파티션에 드라이브 문자 할당 (예: `R:`)
	2. 다음 실행:
		
		```cmd
		vssadmin list shadowstorage /for=R:
		```
	
	→ **존재하면**, 거기에 대한 공간도 고려해야 함  
	→ 존재하지 않으면, 해당 파티션은 VSS 비대상일 수 있음

___

##### 4. 결론

| 판단 항목                                          | 평가                                                          |
| ---------------------------------------------- | ----------------------------------------------------------- |
| C 드라이브에 VSS 저장소 있으니 시스템 예약 파티션 확장은 불필요한 것 아닌가? | 대부분의 경우 맞습니다                                                |
| 하지만 시스템 예약 파티션이 관련될 수 있는 경우                    | **시스템 이미지 백업은 예약 파티션도 포함하므로, VSS 공간이 부족하면 해당 오류 발생**        |
| 따라서 최적 해결은                                     | **C 드라이브의 VSS 공간 조정** + **시스템 예약 파티션 공간 점검**의 **양방향 접근**입니다 |

____

## 결론

### 1. 최종 해결 과정

#### 1. `Workstation` 서비스 복구

해결 과정 2의 전반에서 언급

1. `해결 과정 2` 를 통해 `Workstation` 서비스의 관련 요소들이 손상되어 확인함
2. 1의 이유로 VSS Writer 의 일부가 정상 작동하지 않아 시스템 이미지 생성이 불가했음을 확인
3. 최근 VirtualBox 설치 이후 문제가 발생했으므로 VirtualBox 삭제 방법을 참고하여 불필요한 네트워크 설정을 삭제 및 복구하였음
4. In-place Upgrade 설치를 통해 1의 요소를 복구함

#### 2. 복구 파티션을 임시 비활성화

해결 과정 3의 마지막 부분에서 언급

	1. 시스템 이미지 생성을 위해서는 VSS가 해당 파티션을 백업 대상에 포함하도록 강제하고 있음
	2. 1의 이유로 스냅샷 생성 또는 임시 메타데이터 기록에 실패하여 백업 전체가 중단되었음
	3. 백업 대상에 포함되지 않도록 임시 비활성화 하여 백업 진행이 정상화 되었음

---

### 2. 문제의 원인

#### 1. `Workstation` 서비스의 관련 요소들이 왜 손상되었는가?

- **정황**

| 항목                                        | 내용                             |
| ----------------------------------------- | ------------------------------ |
| 최근 VirtualBox 설치 이후 복구 및 백업 오류 발생         | VirtualBox가 **직접적 원인일 가능성 높음** |
| `Winre.wim`은 611MB로 복구 파티션 거의 포화 상태       | 예전에는 간신히 통과되었지만 지금은 실패         |
| 오류: `0x80780119` / VSS 실패 / ASR Writer 실패 | 복구 파티션 또는 Writer 충돌 의심됨        |

- **VirtualBox 설치가 영향을 줄 수 있는 메커니즘**

| 원인                                                                      | 설명                                                              |
| ----------------------------------------------------------------------- | --------------------------------------------------------------- |
| VirtualBox가 NIC 필터 드라이버 설치                                              | 일부 Windows Writer가 네트워크 관련 서비스(`LanmanWorkstation`, `nsi`)에 의존함 |
| VirtualBox 설치 시 `boot configuration` 또는 `recovery configuration`이 덮어씌워짐 | `Winre.wim` 갱신 또는 위치 재설정 발생 가능                                  |
| 일부 OEM 시스템에서 VirtualBox 설치 시 시스템 예약/복구 파티션의 ID나 레이블이 변경됨                | WinRE가 공간 부족 또는 위치 오류 발생                                        |

- **요약**:
> **VirtualBox 설치 이후** 복구 파티션 내의 WinRE 환경에 **간접적인 변화**가 발생했고,  
> 그로 인해 복구 파티션이 **이전에는 여유가 거의 없지만 통과**,  
> 지금은 **실패하도록 변했을 가능성**이 있다.

#### 2. 시스템 이미지 생성 시 복구 파티션이 문제를 일으키는 이유는?

1번의 결론과 동일

---

## 장기적인 해결 방안

복구 파티션 삭제 후 재생성하고, **WinRE(Windows Recovery Environment)** 위치를 새로 설정해야 함