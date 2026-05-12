# Windows Server Server Core 설치 이후 다음 단계 정리

## 현재 상태

- VMware Workstation Pro 설치 완료
- Windows Server 2022 Evaluation ISO 다운로드 완료
- VMware에 Windows Server VM 생성 완료
- Easy Install 오류 발생 후 수동 설치 방식으로 해결
- Windows Server 2022 Standard Evaluation 설치 완료
- 현재 설치 형태: **Server Core**
- 현재 화면: **SConfig 초기 설정 화면**

---

## 다음 진행 순서

```text
SConfig
→ PowerShell 진입
→ 네트워크 확인
→ 컴퓨터 이름 변경
→ IIS 설치
→ localhost 테스트
→ 호스트 PC에서 VM 웹서버 접속 테스트
```

---

## 1. PowerShell 진입

SConfig 화면에서 아래 번호 입력:

```powershell
15
```

의미:

```text
Exit to command line (PowerShell)
```

---

## 2. 네트워크 확인

PowerShell에서 실행:

```powershell
ipconfig
```

확인할 항목:

```text
IPv4 Address
Subnet Mask
Default Gateway
DNS Servers
```

인터넷 연결 확인:

```powershell
ping 8.8.8.8
```

DNS 확인:

```powershell
ping google.com
```

판단 기준:

| 결과 | 의미 |
|---|---|
| `ping 8.8.8.8` 성공 | 인터넷 경로 정상 |
| `ping google.com` 성공 | DNS 정상 |
| 8.8.8.8은 되는데 google.com 실패 | DNS 문제 가능성 |
| 둘 다 실패 | NAT/네트워크 연결 문제 가능성 |

---

## 3. 컴퓨터 이름 변경

서버 이름을 알아보기 쉽게 변경:

```powershell
Rename-Computer -NewName "WS2022-IIS"
Restart-Computer
```

재부팅 후 다시 SConfig가 나오면 다시 `15`를 입력해 PowerShell로 진입한다.

---

## 4. IIS 설치

PowerShell에서 실행:

```powershell
Install-WindowsFeature Web-Server
```

설치 확인:

```powershell
Get-WindowsFeature Web-Server
```

`Installed`가 보이면 IIS 설치 성공.

---

## 5. 서버 내부에서 IIS 접속 테스트

```powershell
Invoke-WebRequest http://localhost
```

또는:

```powershell
curl http://localhost
```

HTML 응답이 나오면 IIS가 정상 작동 중이다.

---

## 6. VM IP 확인

```powershell
ipconfig
```

예시:

```text
IPv4 Address . . . . . . . . . . . : 192.168.xxx.xxx
```

이 IP를 호스트 PC 브라우저 접속 테스트에 사용한다.

---

## 7. 호스트 PC에서 접속 테스트

실제 Windows PC 브라우저에서 접속:

```text
http://VM_IP주소
```

예시:

```text
http://192.168.100.128
```

주의:

- 현재 네트워크가 NAT라면 호스트 PC에서 VM 웹서버로 바로 접속이 안 될 수도 있다.
- 접속이 안 되면 나중에 VMware 네트워크 설정을 **Bridged**로 바꿔 테스트할 수 있다.

---

## 오늘의 최소 목표

```text
1. SConfig에서 15 입력
2. ipconfig 확인
3. ping 8.8.8.8 실행
4. ping google.com 실행
5. Install-WindowsFeature Web-Server 실행
6. curl http://localhost 실행
```

이 단계까지 완료하면 Windows Server Core에서 IIS 웹서버 설치와 기본 동작 확인까지 끝난다.

---

# 있었던 질의
# VMware Workstation / Windows Server / IIS 질의 기록

> 목적: IT공부 프로젝트 내에서 VMware Workstation, Windows Server, IIS 실습 중 나온 질문과 답변을 간단히 기록한다.

---

## 1. `Rename-Computer`는 왜 쓰는가?

### 질문
- `Rename-Computer`은 왜 쓰는거야?

### 답변 요약
- `Rename-Computer`는 Windows 컴퓨터 이름을 바꾸는 PowerShell 명령어다.
- Windows Server 설치 직후 기본 이름은 `WIN-...` 형태라 기억하기 어렵다.
- 서버 역할에 맞게 `WEB01`, `DB01`, `DC01`처럼 바꾸면 관리가 쉽다.
- IIS 설치 자체에 필수는 아니지만, 실무식 서버 세팅에서는 초반에 자주 하는 기본 작업이다.

예시:

```powershell
Rename-Computer -NewName "WEB01" -Restart
```

---

## 2. 컴퓨터 이름은 나중에 어디에 쓰이는가?

### 질문
- 컴퓨터 이름은 나중에 어디에 쓰이는거야?

### 답변 요약
컴퓨터 이름은 서버를 식별할 때 쓰인다.

주요 사용처:

- 네트워크에서 서버 구분
- 원격 접속
- 공유 폴더 접근
- Active Directory / 도메인 환경에서 장비 식별
- 이벤트 로그, 보안 로그, 모니터링, 장애 분석

예시:

```text
\\WEB01\share
WEB01.company.local
```

정리:

> 컴퓨터 이름은 단순한 별명이 아니라, Windows 관리 환경에서 서버를 구분하는 공식 식별자에 가깝다.

---

## 3. 컴퓨터 이름과 DNS 별칭의 관계

### 질문
- 보통 꼭 DNS 별칭으로 쓰이진 않는다는데 윈도우 "서버"여서 식별자로 쓰는거야? 이건 나중에 알게되나

### 답변 요약
- 맞다. 꼭 DNS 별칭으로 쓰려고 바꾸는 것은 아니다.
- Windows Server를 관리 대상 장비로 식별하기 위해 이름을 정리하는 의미가 더 크다.

구분:

| 개념 | 의미 |
|---|---|
| 컴퓨터 이름 | Windows가 자기 자신을 식별하는 이름 |
| DNS 이름 | 이름을 IP로 해석하기 위한 네트워크 이름 |
| 서버 역할 이름 | 사람이 관리하기 쉽게 붙이는 이름 |

정리:

> 지금은 `Rename-Computer`가 DNS 공부용이라기보다, Windows Server를 관리하기 쉽게 만드는 기본 설정이라고 이해하면 된다.

---

## 4. VMware 화면 전환이 불편한 문제

### 질문
- `Ctrl + Alt`로 탈출하고, `Alt + Tab`으로 GPT 갔다가 다시 화면 클릭해서 진입하는 게 너무 귀찮다.

### 답변 요약
가장 좋은 해결책은 **VMware Tools 설치**다.

VMware Tools 설치 시 기대 효과:

- 마우스가 VM 안팎으로 자연스럽게 이동
- `Ctrl + Alt` 탈출 사용 감소
- 화면 해상도 자동 조절
- 복사/붙여넣기 개선 가능

메뉴:

```text
VM → Install VMware Tools
```

또는 이미 설치 모드가 시작되어 있으면:

```text
VM → Cancel VMware Tools Installation
```

이 보일 수 있다.

---

## 5. VMware Tools 설치 화면 확인

### 질문
- 메뉴에 `Cancel VMware Tools Installation`이 보인다. 이게 뭐냐?

### 답변 요약
- `Cancel VMware Tools Installation`이 보인다는 것은 VMware Tools 설치용 가상 CD가 이미 게스트 OS에 삽입된 상태라는 뜻이다.
- 이때는 VMware 메뉴를 다시 누르는 게 아니라 Windows Server 안에서 설치 파일을 실행해야 한다.

PowerShell에서 확인:

```powershell
Get-Volume
```

드라이브 예시:

```text
D: VMware Tools
```

설치 실행:

```powershell
D:
dir
.\setup64.exe
```

또는:

```powershell
.\setup.exe
```

설치 옵션은 보통 `Typical` 선택 후 재부팅하면 된다.

---

## 6. `Web-Server`는 무엇인가?

### 질문
- `Web-Server` 이 뭐야?

### 답변 요약
`Web-Server`는 Windows Server에서 IIS 웹 서버 역할의 내부 이름이다.

예시:

```powershell
Install-WindowsFeature Web-Server
```

관계:

| 표현 | 의미 |
|---|---|
| `Web-Server` | PowerShell에서 쓰는 기능/역할 이름 |
| `Web Server (IIS)` | Server Manager에서 보이는 표시 이름 |
| IIS | 실제 웹 서버 역할 이름 |

확인:

```powershell
Get-WindowsFeature Web-Server
```

정리:

> `Web-Server`는 IIS를 설치하거나 조회할 때 쓰는 Windows Server 기능 이름이다.

---

## 7. `curl http://localhost` 오류와 IE 초기 설정

### 질문
- `curl http://localhost` 실행 시 Internet Explorer 관련 오류가 나왔다.
- Internet Explorer 초기 설정이 뭐야?

### 답변 요약
Windows PowerShell에서 `curl`은 실제 Linux curl이 아니라 보통 아래 명령어의 별칭이다.

```powershell
Invoke-WebRequest
```

옛 PowerShell의 `Invoke-WebRequest`는 Internet Explorer의 HTML 파싱 엔진을 참조하는 경우가 있다.

IE를 한 번도 실행하지 않았거나 초기 설정이 끝나지 않으면 다음과 같은 오류가 날 수 있다.

```text
Internet Explorer's first-launch configuration is not complete
```

해결:

```powershell
Invoke-WebRequest http://localhost -UseBasicParsing
```

또는:

```powershell
curl http://localhost -UseBasicParsing
```

응답 코드만 확인:

```powershell
Invoke-WebRequest http://localhost -UseBasicParsing | Select-Object StatusCode
```

`StatusCode`가 `200`이면 IIS가 정상 응답하는 상태다.

---

## 8. IE는 나중에 브라우저처럼 쓰는가?

### 질문
- IE는 나중에 브라우저처럼 쓰는거야?

### 답변 요약
- 거의 안 쓴다고 보면 된다.
- Internet Explorer는 예전 Windows 기본 브라우저다.
- 현재는 Microsoft Edge가 기본 브라우저다.
- Windows Server에 IE 관련 구성 요소가 남아 있어 PowerShell 오류에서 언급된 것이다.

정리:

> 지금 오류는 IE를 직접 써야 한다는 뜻이 아니라, PowerShell이 오래된 방식으로 웹 응답을 읽으려다 막힌 것이다.

---

## 9. 호스트 PC `ipconfig` 결과 해석

### 질문
호스트 Windows PC의 `ipconfig` 결과를 해석해달라고 요청했다.

주요 결과:

```text
vEthernet (WSL): 172.24.96.1
VMware Network Adapter VMnet1: 192.168.209.1
VMware Network Adapter VMnet8: 192.168.223.1
Wi-Fi: 192.168.45.220
기본 게이트웨이: 192.168.45.1
```

### 답변 요약
현재 실제 인터넷 연결은 Wi-Fi다.

| 어댑터 | IP | 의미 |
|---|---:|---|
| `vEthernet (WSL)` | `172.24.96.1` | WSL2용 가상 네트워크 |
| `VMnet1` | `192.168.209.1` | VMware Host-only 네트워크 |
| `VMnet8` | `192.168.223.1` | VMware NAT 네트워크 |
| `Wi-Fi` | `192.168.45.220` | 실제 인터넷 연결 |
| 기본 게이트웨이 | `192.168.45.1` | 공유기 / 라우터 |

IIS 서버 접속 주소는 호스트 PC가 아니라 **VM 안의 Windows Server에서 `ipconfig`를 실행해서 확인해야 한다.**

---

## 10. 공유기, PC, VMware 가상 네트워크 구조

### 질문
- `192.168.45.1` 라우터 안에 `192.168.45.220` 내 PC가 있고, 그 안에 VMware의 `192.168.209.1`, `192.168.223.1`이 존재하는 구조냐?
- VMnet1, VMnet8은 실제 노드가 아니라 Host-only와 NAT용 라우터냐?

### 답변 요약
거의 맞지만, 표현을 조금 수정하면 다음과 같다.

구조:

```text
공유기 / 라우터
192.168.45.1
   │
   │ Wi-Fi 네트워크
   │
내 PC 실제 Wi-Fi
192.168.45.220
   │
   ├─ VMware 가상 네트워크 VMnet1
   │   └─ Host-only 대역: 192.168.209.0/24
   │
   └─ VMware 가상 네트워크 VMnet8
       └─ NAT 대역: 192.168.223.0/24
```

중요한 점:

- `192.168.209.1`, `192.168.223.1`은 VM 내부 IP가 아니다.
- 호스트 PC에 만들어진 VMware용 가상 네트워크 어댑터 IP다.
- `VMnet1`, `VMnet8`은 실제 물리 장비가 아니라 VMware가 만든 가상 네트워크 인터페이스다.

NAT 흐름:

```text
VM 게스트
192.168.223.x
   │
VMnet8 가상 네트워크
   │
호스트 VMware NAT 기능
   │
호스트 Wi-Fi
192.168.45.220
   │
공유기
192.168.45.1
   │
인터넷
```

Host-only 흐름:

```text
호스트 PC
192.168.209.1
   │
VMnet1 가상 네트워크
   │
VM 게스트
192.168.209.x
```

정리:

> 네 PC는 실제 Wi-Fi 네트워크에도 붙어 있고, 동시에 VMware가 만든 가상 네트워크 2개에도 붙어 있다.

---

## 11. WSL / Ubuntu와 `172.24.96.1`

### 질문
- 내부적으로 WSL/Ubuntu를 쓰면 `172.24.96.1` 쪽으로 연결되는 거냐?

### 답변 요약
맞다. WSL2/Ubuntu는 보통 Windows가 만든 WSL용 가상 네트워크를 통해 연결된다.

구조:

```text
Windows 호스트
172.24.96.1
   │
vEthernet (WSL)
   │
WSL2 Ubuntu
172.24.xxx.xxx
```

중요:

- `172.24.96.1`은 Ubuntu의 IP가 아니다.
- Windows 호스트 쪽에 생긴 WSL용 가상 어댑터 IP다.
- Ubuntu 내부 IP는 WSL 안에서 따로 확인해야 한다.

WSL Ubuntu에서 확인:

```bash
hostname -I
```

또는:

```bash
ip addr
```

인터넷 흐름:

```text
Ubuntu WSL
172.24.xxx.xxx
   │
Windows WSL NAT / 가상 네트워크
172.24.96.1
   │
Windows Wi-Fi
192.168.45.220
   │
공유기
192.168.45.1
   │
인터넷
```

정리:

> WSL Ubuntu는 실제 공유기에 직접 붙은 것이 아니라, Windows가 만든 `vEthernet (WSL)` 가상 네트워크 안에서 돌고 Windows를 통해 외부로 나간다.

---

## 전체 요약

이번 세션에서는 다음 흐름을 다뤘다.

1. Windows Server의 컴퓨터 이름 변경 목적
2. 컴퓨터 이름과 DNS/관리 식별자의 차이
3. VMware Tools 설치 목적과 설치 방법
4. IIS 역할 이름 `Web-Server`의 의미
5. PowerShell `curl` / `Invoke-WebRequest`와 IE 초기 설정 오류
6. 호스트 PC의 `ipconfig` 결과 해석
7. VMware VMnet1 / VMnet8 구조
8. WSL2 가상 네트워크 구조

