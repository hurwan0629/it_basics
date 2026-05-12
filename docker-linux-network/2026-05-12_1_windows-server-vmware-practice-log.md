# Windows Server / VMware 설치 실습 기록

## 목적

Windows Server를 VMware 가상머신에 설치하고, 이후 IIS 웹서버 실습까지 진행하기 위한 환경을 구성했다.

---

## 1. Windows Server 개념 질문

### 질문
- “윈도우 서버가 뭐야?”

### 답변 요약
- Windows Server는 Microsoft의 서버용 운영체제라고 설명했다.
- 일반 Windows 10/11이 개인 PC용이라면, Windows Server는 회사나 기관에서 사용자, 컴퓨터, 파일, 웹서비스, 네트워크를 관리하기 위한 OS라고 설명했다.
- 주요 역할로 다음을 설명했다.
  - 사용자/계정 관리
  - 파일 서버
  - 웹 서버
  - DB 서버
  - DNS/DHCP 서버
- Linux Server와 Windows Server의 차이도 비교했다.
- 실습 순서로 다음을 제안했다.
  - Windows Server 설치
  - RDP 사용
  - IIS 설치
  - 방화벽/포트 확인
  - 공유 폴더
  - 사용자 계정/권한
  - Active Directory 맛보기

---

## 2. Windows Server 실습 예시 요청

### 질문
- “이거 실습 예시 하나 보여줄래? 까는거부터.”

### 답변 요약
- 실습 목표를 다음으로 잡았다.

```text
Windows Server 설치
→ IIS 설치
→ index.html 만들기
→ 브라우저 접속 확인
```

- 준비물로 다음을 안내했다.
  - 가상머신 프로그램
  - Windows Server ISO
  - PC 사양
- Windows Server 2022 Evaluation ISO를 추천했다.
- 처음에는 GUI가 있는 `Desktop Experience`를 추천했다.
- IIS 설치 방법을 GUI와 PowerShell 두 방식으로 설명했다.
- IIS 기본 웹 루트 경로를 설명했다.

```text
C:\inetpub\wwwroot
```

---

## 3. VirtualBox 개념 질문

### 질문
- “하나씩 해보자. VirtualBox가 뭐야?”

### 답변 요약
- VirtualBox는 내 컴퓨터 안에 가상 컴퓨터를 만드는 프로그램이라고 설명했다.
- 가상머신 프로그램이라고 설명했다.
- 실제 PC에 Windows Server를 바로 설치하지 않고, VirtualBox 안에 설치하면 안전하다고 설명했다.
- Docker와 비교해서 설명했다.
  - Docker는 컨테이너
  - VirtualBox는 가상 컴퓨터

---

## 4. VirtualBox 무게 질문

### 질문
- “VirtualBox는 얼마나 무거워? 프로그램이”

### 답변 요약
- VirtualBox 자체는 무겁지 않고, 그 안에서 실행하는 VM이 무겁다고 설명했다.
- Windows Server GUI VM 기준으로 RAM 4GB 이상을 권장했다.
- Docker, Chrome, VSCode와 함께 쓰면 RAM 사용량이 커질 수 있다고 설명했다.

---

## 5. VMware와 VirtualBox 차이 질문

### 질문
- “vmware하고 뭐가 달라?”

### 답변 요약
- VirtualBox와 VMware 모두 가상머신 프로그램이라고 설명했다.
- VMware가 성능과 안정성 면에서 더 부드럽게 느껴지는 경우가 많다고 설명했다.
- Windows Server + IIS + 네트워크 실습 기준으로는 VMware를 1순위로 추천했다.
- Docker Desktop과 VMware가 모두 가상화 기능을 쓰기 때문에 충돌 또는 성능 저하 가능성이 있다고 설명했다.

---

## 6. VMware winget 설치 시도

### 질문
- “vmware 까는법 알려줘. 가능하면 winget 쓰고싶어”

### 답변 요약
- winget으로 VMware Workstation Pro를 설치하는 명령어를 안내했다.

```powershell
winget search vmware
winget install --id VMware.WorkstationPro --exact
```

---

## 7. winget 검색 결과 문제

### 사용자가 한 것
- `winget search vmware`를 실행했다.
- 결과에 VMware Workstation Pro가 나오지 않았다.
- 다음과 같은 패키지들이 보였다.
  - ControlUp Remote DX
  - Hyper-V-Switch
  - RingCentral Phone VMware
  - RVTools
  - DiskInternals VMFS Recovery
  - Spring Tools for Eclipse

### 답변 요약
- 현재 winget 검색 결과에 `VMware.WorkstationPro`가 안 보이는 상태라고 설명했다.
- 다음 명령어를 안내했다.

```powershell
winget source update
winget search --id VMware.WorkstationPro --exact
winget search "VMware Workstation"
winget install -e --id VMware.WorkstationPro --source winget
```

---

## 8. winget 소스 리셋 및 검색 실패

### 사용자가 한 것
- 관리자 PowerShell에서 다음을 실행했다.

```powershell
winget source reset --force
winget source update
winget source list
winget search "VMware Workstation"
```

- `winget`, `msstore`, `winget-font` 소스가 정상적으로 표시되었다.
- 그래도 `VMware Workstation` 검색 결과가 나오지 않았다.

### 질문
- “포기 해야하나”

### 답변 요약
- winget 설치는 포기해도 된다고 답했다.
- winget 자체 문제나 소스 설정 문제는 아니고, 현재 저장소 검색에서 VMware Workstation 패키지가 잡히지 않는 상황이라고 설명했다.
- 선택지로 다음을 제시했다.
  - 공식 설치파일로 VMware 설치
  - VirtualBox로 진행
  - winget 계속 시도는 비추천
- VMware를 공식 Broadcom/VMware 사이트에서 받는 방법을 안내했다.
- VirtualBox 설치 명령어도 대안으로 제시했다.

```powershell
winget install -e --id Oracle.VirtualBox
```

---

## 9. Broadcom VMware 다운로드 페이지 선택

### 사용자가 한 것
- Broadcom 사이트의 VMware Workstation Pro 다운로드 페이지에 접속했다.
- 화면에는 다음 항목들이 있었다.
  - VMware Workstation Pro 25H2 for Windows
  - VMware Workstation Pro 25H2 for Linux
  - VMware Workstation Pro 17.0 for Windows
  - VMware Workstation Pro 17.0 for Linux

### 질문
- “여기에서 뭘 깔면 돼?”

### 답변 요약
- Windows PC에서 실습할 것이므로 `VMware Workstation Pro 25H2 for Windows`를 선택하라고 안내했다.
- Linux용과 17.0 구버전은 선택하지 않아도 된다고 설명했다.

---

## 10. VMware release 선택

### 사용자가 한 것
- `25H2u1`과 `25H2`가 보이는 화면을 보여줬다.

### 질문
- “뭐골라?”

### 답변 요약
- `25H2u1`을 고르라고 안내했다.
- `u1`은 Update 1 의미로, 같은 25H2 계열이면 더 최신 패치 버전이라고 설명했다.

---

## 11. Broadcom 라이선스 페이지 질문

### 질문
- `https://www.broadcom.com/company/legal/licensing`
- “이거 설명해줘”

### 답변 요약
- 해당 페이지는 Broadcom 제품을 쓰기 전에 적용되는 라이선스/서비스 약관 모음 페이지라고 설명했다.
- VMware가 Broadcom에 인수된 뒤 VMware 제품 관련 약관도 Broadcom 쪽에서 관리된다고 설명했다.
- 다운로드 페이지라기보다는 라이선스 조건과 서비스 약관 저장소에 가깝다고 설명했다.

---

## 12. 약관에서 신경 쓸 점 질문

### 질문
- “내말은 이거 약관내용하고 내가 신경쓸게 잇는지”

### 답변 요약
- 개인 PC에서 Windows Server/IIS 실습용으로 VMware Workstation Pro를 설치하는 정도라면 크게 신경 쓸 것은 많지 않다고 설명했다.
- 신경 쓸 점으로 다음을 정리했다.
  - 개인/교육/상업 용도로 무료 사용 가능
  - 라이선스 키 필요 없음
  - Broadcom 계정 정보와 Trade Compliance 정보는 정확히 입력
  - 설치 파일을 임의로 재배포하지 않는 것이 안전
  - 회사/기관 대량 배포나 업무 표준 환경이면 약관 확인 필요

---

## 13. Broadcom Trade Compliance 입력

### 사용자가 한 것
- Trade Compliance and Download Conditions 입력 화면을 보여줬다.
- Company 필드가 필수였다.

### 질문
- “기업은 뭘 적어야해?”

### 답변 요약
- 개인 공부용이면 Company에는 다음 중 하나를 넣으면 된다고 설명했다.

```text
Individual
Personal
```

- 추천은 `Individual`이라고 안내했다.
- 허위 회사명보다 개인 사용자임을 나타내는 값이 낫다고 설명했다.

---

## 14. VMware 설치파일 보관 여부

### 질문
- “다운받은 exe 설치파일은 설치 후에도 가지고 있는게 좋을까”

### 답변 요약
- 설치가 정상적으로 끝났고 VMware가 실행된다면 삭제해도 된다고 설명했다.
- 다만 다음 경우에는 잠깐 보관해도 된다고 말했다.
  - 재설치할 수도 있음
  - 다시 받기 귀찮음
  - 같은 버전으로 나중에 다시 설치하고 싶음
  - Broadcom 다운로드 과정이 번거로움
- VMware 실행 확인 후 삭제해도 된다고 정리했다.

---

## 15. VMware 설치 중 User Experience Settings

### 사용자가 한 것
- VMware Workstation Pro Setup의 User Experience Settings 화면을 보여줬다.
- 다음 체크박스가 있었다.
  - Check for product updates on startup
  - Join the VMware Customer Experience Improvement Program

### 질문
- “이건 뭐야?”

### 답변 요약
- 업데이트 자동 확인과 고객 경험 개선 프로그램 참여 여부를 묻는 화면이라고 설명했다.
- 추천 설정은 다음과 같았다.

```text
Check for product updates on startup → 체크 유지
Join the VMware Customer Experience Improvement Program → 체크 해제 추천
```

---

## 16. Docker와 VMware 충돌 가능성 질문

### 질문
- “깔리는 중이고 뭐 하나 물어보자면 아까 말한 도커와 vmware 충돌 가능성 원인이 뭐야?”

### 답변 요약
- Docker Desktop과 VMware가 둘 다 CPU 가상화 기능을 쓰기 때문이라고 설명했다.
- Docker Desktop은 Windows에서 보통 WSL2/Hyper-V 기반 가상 환경을 사용한다고 설명했다.
- VMware는 별도의 VM을 실행하기 위해 CPU 가상화 기능을 쓴다고 설명했다.
- 충돌 또는 성능 저하 원인으로 다음을 설명했다.
  - Hyper-V 계층 문제
  - RAM/CPU/디스크 I/O 자원 경쟁
  - 가상 네트워크 어댑터가 많아져 혼란 발생
- Docker 실습할 때는 VMware VM을 끄고, Windows Server 실습할 때는 Docker Desktop을 끄는 것을 추천했다.

---

## 17. Hyper-V, VM, CPU 가상화 질문

### 질문
- “HyperV, VM, cpu 가상화가 뭔지 알려줘”

### 답변 요약
- 용어를 다음처럼 정리했다.

```text
CPU 가상화
→ 가상머신을 가능하게 하는 CPU 기능

VM
→ 가상으로 만든 컴퓨터

Hypervisor
→ VM을 만들고 관리하는 프로그램/계층

Hyper-V
→ Microsoft가 만든 Hypervisor
```

- VM은 내 컴퓨터 안에 만든 가상 컴퓨터라고 설명했다.
- CPU 가상화는 Intel VT-x, AMD-V 같은 기능으로 설명했다.
- Hyper-V는 Windows의 공식 가상머신 관리자라고 설명했다.
- Docker Desktop은 WSL2를 통해 내부적으로 가상화 기술을 쓴다고 설명했다.

---

## 18. VMware 설치 완료 후 첫 화면

### 사용자가 한 것
- VMware Workstation Pro 25H2가 실행된 화면을 보여줬다.
- 화면에는 다음 버튼이 있었다.
  - Create a New Virtual Machine
  - Open a Virtual Machine
  - Connect to a Remote Server

### 질문
- “여기에서 이제 뭐할까”

### 답변 요약
- 이제 Windows Server ISO를 받아서 새 가상머신을 만들면 된다고 설명했다.
- Windows Server 2022 Evaluation ISO를 추천했다.
- VMware에서 `Create a New Virtual Machine`을 누르라고 안내했다.

---

## 19. Windows Server 평가판 등록 폼

### 사용자가 한 것
- Microsoft Windows Server 2022 평가판 등록 페이지를 보여줬다.
- 이름, 성, 회사 이메일, 회사 이름, 직원 수, 직함, 회사 전화, 국가/지역 등을 입력하는 화면이었다.
- Company Name에 `Individual`을 입력했다.

### 질문
- “여기에서 사진처럼 하면 되나”

### 답변 요약
- 대체로 그렇게 해도 된다고 답했다.
- 이름/성은 영문 기준으로 `Wan / Hur`가 더 자연스럽다고 안내했다.
- 회사 이름은 `Individual`이 적절하다고 설명했다.
- 직함은 `Student`가 적절하다고 설명했다.
- 마케팅 수신 동의는 선택 항목이면 체크하지 않는 것을 권장했다.

---

## 20. ISO와 Azure 질문

### 질문
- “ISO와 Azure에 대해 알려줘.”

### 답변 요약
- ISO는 운영체제 설치용 파일을 하나로 묶어둔 디스크 이미지 파일이라고 설명했다.
- 예전 Windows 설치 DVD 내용을 파일 하나로 만든 것과 같다고 설명했다.
- 이번 실습에서는 ISO를 VMware에 넣어 가상 DVD처럼 사용한다고 설명했다.
- Azure는 Microsoft의 클라우드 서비스라고 설명했다.
- ISO 방식과 Azure 방식 차이를 비교했다.
  - ISO: 내 PC의 VMware에 직접 설치
  - Azure: Microsoft 클라우드에 서버 생성
- 지금 실습에는 ISO가 맞다고 설명했다.

---

## 21. ISO 불법 여부와 Azure 실제 사용 여부 질문

### 질문
1. “iso로 개인 불법 다운로드 가능성은 없어? 윈도우 유료 아니야?”
2. “Azure는 실제 회사에서 사용해?”

### 답변 요약
- Microsoft 공식 페이지에서 받는 Windows Server Evaluation ISO는 불법 다운로드가 아니라고 설명했다.
- Windows Server 정식 운영은 유료지만, 평가판은 테스트/학습용으로 제공된다고 설명했다.
- 크랙, 인증 우회, 불법 KMS 인증은 하면 안 된다고 설명했다.
- Azure는 실제 회사에서 많이 사용하는 클라우드 플랫폼이라고 설명했다.
- 회사에서 Azure를 사용하는 예로 다음을 들었다.
  - 서버 VM 운영
  - 웹 애플리케이션 배포
  - 데이터베이스
  - 스토리지
  - AI 서비스
  - 백업/재해복구
  - 모니터링/보안

---

## 22. ISO 다운로드 중 다음 과정 안내

### 질문
- “지금 일단 iso 깔리는 중이고 깔리면 뭘 하게될지 알려줘.”

### 답변 요약
- ISO 다운로드 후 진행 흐름을 설명했다.

```text
1. Windows Server ISO 다운로드 완료
2. VMware에서 새 가상머신 생성
3. ISO를 가상 DVD처럼 연결
4. Windows Server 설치
5. 관리자 비밀번호 설정
6. Windows Server 첫 로그인
7. 기본 네트워크 확인
8. IIS 설치
9. 웹페이지 접속 테스트
```

---

## 23. VM 네트워크 설정, Windows Server 역할, IIS 질문

### 질문
1. VM 네트워크 설정방식
2. Windows Server 역할
3. IIS

### 답변 요약
- VM 네트워크 방식으로 NAT, Bridged, Host-only를 설명했다.
- Windows Server 역할은 서버 기능을 Role로 추가해서 쓰는 구조라고 설명했다.
- 대표 역할로 다음을 설명했다.
  - Web Server IIS
  - File and Storage Services
  - Active Directory Domain Services
  - DNS Server
  - DHCP Server
  - Remote Desktop Services
  - Hyper-V
- IIS는 Microsoft의 웹서버이며, Linux의 Apache/Nginx와 비슷하다고 설명했다.
- IIS 기본 웹 루트는 다음이라고 설명했다.

```text
C:\inetpub\wwwroot
```

---

## 24. NAT와 Bridged 이해 확인

### 질문
- “NAT는 VM 랜카드를 호스트 인터넷 인터페이스에 연결하는거고 Bridged는 VM랜카드를 직접 라우터에 연결하는거야?”

### 답변 요약
- 거의 맞다고 답했다.
- NAT는 정확히 다음 구조라고 설명했다.

```text
VM 랜카드
→ VMware의 가상 NAT 네트워크
→ 호스트 PC
→ 호스트의 실제 인터넷 인터페이스
→ 공유기/인터넷
```

- Bridged는 VM 랜카드를 호스트의 실제 랜카드/Wi-Fi에 브리지로 붙여서 공유기 입장에서 별도 컴퓨터처럼 보이게 하는 방식이라고 설명했다.

---

## 25. 네트워크 인터페이스 개념 질문

### 질문
- “네트워크에는 뭐 인터페이스라는 정보는 현재 장치와 연결된 기기들을 뜻했었나?”

### 답변 요약
- 네트워크 인터페이스는 연결된 기기들이 아니라, 현재 장치가 네트워크에 연결하기 위한 출입구/연결 지점이라고 설명했다.
- 예시로 다음을 들었다.
  - Wi-Fi 어댑터
  - 유선 LAN 어댑터
  - WSL 가상 어댑터
  - Docker 가상 어댑터
  - VMware Network Adapter VMnet1
  - VMware Network Adapter VMnet8
- Linux의 `eth0`, Windows의 `Wi-Fi`, `VMware Network Adapter VMnet8`, `vEthernet (WSL)` 등을 인터페이스 예시로 설명했다.

---

## 26. VM에서 라우터까지 홉 질문

### 질문
- “그러면 VM에서 라우터까지 가는데 2홉(TTL) 필요한건가 (내 컴퓨터가 공유기랑 연결되었다면)”

### 답변 요약
- 설정 방식에 따라 다르다고 설명했다.
- TTL은 라우터처럼 L3에서 패킷을 다른 네트워크로 전달할 때 줄어든다고 설명했다.
- Bridged에서는 VM이 공유기 네트워크에 직접 붙은 것처럼 동작하므로 공유기까지는 같은 LAN 통신이라고 설명했다.
- NAT에서는 VMware NAT Gateway가 중간에 있으므로 인터넷 목적지 기준으로는 VMware NAT Gateway와 공유기를 거칠 수 있다고 설명했다.
- 확인 명령어로 `tracert 8.8.8.8`을 안내했다.

---

## 27. ISO 다운로드 완료 후 VM 생성

### 질문
- “이제 iso 다운됏어”

### 답변 요약
- VMware에서 새 VM을 만들라고 안내했다.
- 원래는 Typical 기준으로 설명했으나, 사용자가 Custom이 더 낫지 않냐고 질문했다.

---

## 28. Typical vs Custom 질문

### 질문
- “typical대신 custom으로 하는게 더 자세히 아는데 좋지 않아?”

### 답변 요약
- 공부 목적이면 `Custom`이 더 좋다고 답했다.
- `Typical`은 빠르게 만들기용이고, `Custom`은 하드웨어/펌웨어/디스크/네트워크 구조를 직접 볼 수 있다고 설명했다.
- Custom에서 보게 될 설정을 설명했다.
  - Hardware Compatibility
  - OS 설치 방식
  - Guest OS 종류
  - VM 이름/저장 위치
  - Firmware Type
  - CPU
  - RAM
  - Network Type
  - I/O Controller
  - Disk Type
  - Virtual Disk
  - ISO

---

## 29. ISO 선택 오류

### 사용자가 한 것
- VMware Custom 마법사에서 ISO 선택 화면을 보여줬다.
- 선택된 경로가 VMware 설치 폴더 내부의 `windows.iso`처럼 보였다.

### 질문
- “이게 무슨말이야?”

### 답변 요약
- VMware가 ISO 안의 운영체제를 자동으로 판별하지 못했다는 뜻이라고 설명했다.
- 하지만 더 중요한 문제는 선택된 ISO가 Windows Server 설치 ISO가 아니라 VMware Tools 관련 ISO일 가능성이 높다고 설명했다.
- 다운로드 폴더에서 Microsoft에서 받은 Windows Server ISO를 선택하라고 안내했다.

---

## 30. VMware Easy Install 화면

### 사용자가 한 것
- Easy Install Information 화면을 보여줬다.
- Windows Server 2022로 인식되었다.
- 제품 키, 설치 버전, 이름, 비밀번호를 입력하는 화면이었다.

### 질문
- “이건 뭐야? iso 잘 넣엇어”

### 답변 요약
- ISO는 제대로 인식된 상태라고 답했다.
- VMware가 Windows Server 2022 설치 ISO로 인식했고, Easy Install 설정을 받는 화면이라고 설명했다.
- 제품 키는 평가판이면 비워도 된다고 설명했다.
- Standard를 추천했지만 Datacenter도 실습 가능하다고 설명했다.
- Easy Install은 편하지만 설치 과정을 직접 보는 경험은 줄어든다고 설명했다.

---

## 31. 제품 키 없음 경고

### 사용자가 한 것
- 제품 키를 입력하지 않았다는 경고창을 보여줬다.

### 질문
- “이건 무슨뜻이야?”

### 답변 요약
- 제품 키 없이 설치는 가능하지만 나중에 정품 인증해야 한다는 의미라고 설명했다.
- 평가판 ISO 실습 중이면 계속 진행해도 된다고 안내했다.
- 체크박스는 굳이 선택하지 않아도 된다고 설명했다.

---

## 32. VM 이름과 경로 설정

### 사용자가 한 것
- VM 이름과 저장 경로 설정 화면을 보여줬다.
- 기본 이름은 `Windows Server 2022`였다.
- 경로가 OneDrive 아래였다.

### 질문
- “이름은 그대로 가면 되나? 경로는 OneDrive말고 내 C:\Document\VMware\ 로 해도 되나”

### 답변 요약
- 이름은 그대로도 가능하지만 실습 목적이 보이게 바꾸는 것을 추천했다.

```text
Windows Server 2022 IIS Practice
```

- OneDrive 경로는 피하는 것이 좋다고 설명했다.
- VM 파일은 크고 자주 변경되므로 OneDrive 동기화와 충돌하거나 느려질 수 있다고 설명했다.
- 추천 경로를 제시했다.

```text
C:\VMs\WindowsServer2022-IIS
```

---

## 33. UEFI / BIOS 화면

### 사용자가 한 것
- Firmware type 선택 화면을 보여줬다.
- BIOS와 UEFI가 있었고 UEFI가 선택되어 있었다.
- Secure Boot는 체크되어 있지 않았다.

### 질문
- “이건 뭐야?”

### 답변 요약
- VM이 부팅할 때 사용할 펌웨어 방식을 고르는 화면이라고 설명했다.
- BIOS는 오래된 방식, UEFI는 현대적인 부팅 시스템이라고 설명했다.
- Windows Server 2022는 UEFI가 자연스럽다고 설명했다.
- Secure Boot는 실습용에서는 꺼도 된다고 설명했다.
- 현재처럼 UEFI 선택, Secure Boot 해제로 진행하라고 안내했다.

---

## 34. CPU 설정

### 사용자가 한 것
- Processor Configuration 화면을 보여줬다.
- 현재 `Number of processors: 2`, `Number of cores per processor: 1`이었다.
- 사용자는 PC가 RAM 32GB, i9 24코어라고 말했다.

### 질문
- “내가 32 메모리 i9 24코어면 어떻게 하는게 좋을까”

### 답변 요약
- 실습용 VM에 너무 많이 줄 필요는 없다고 설명했다.
- 추천값은 다음이었다.

```text
Number of processors: 1
Number of cores per processor: 4
Total processor cores: 4
```

- 2 processors × 1 core보다 1 processor × 4 cores가 일반적인 실습용 서버 구조에 더 자연스럽다고 설명했다.

---

## 35. 메모리 설정

### 사용자가 한 것
- Memory for the Virtual Machine 화면을 보여줬다.
- 현재 2048MB가 설정되어 있었다.

### 질문
- “이건 4Gb면 적당한가?”

### 답변 요약
- RAM 32GB인 PC에서는 4GB도 가능하지만 8GB를 추천한다고 답했다.
- 추천값은 다음이었다.

```text
8192 MB
```

---

## 36. NAT와 Bridged 선택

### 질문
- “이거 NAT와 브릿지중 뭐가 나아?”

### 답변 요약
- 처음 Windows Server 설치 단계에서는 NAT를 추천했다.
- 설치, 업데이트, 기본 설정, IIS 설치까지는 NAT가 안전하고 단순하다고 설명했다.
- 나중에 실제 PC 브라우저에서 VM 웹서버로 접속할 때 Bridged로 바꿔도 된다고 설명했다.

---

## 37. 네트워크 설정 변경 가능 여부

### 질문
- “나중에 변경 가능한거지?”

### 답변 요약
- 나중에 변경 가능하다고 답했다.
- VM을 완전히 종료한 뒤 다음 경로에서 바꾸면 된다고 설명했다.

```text
Edit virtual machine settings
→ Network Adapter
→ NAT / Bridged / Host-only 선택
```

---

## 38. 다음 스텝 전체 설명

### 질문
- “그 다음 스텝들 다 알려줄래? 설명하고”

### 답변 요약
- NAT 선택 이후부터 Windows Server 설치 완료, IIS 실습까지 전체 흐름을 설명했다.
- 이후 설정으로 다음을 안내했다.
  - I/O Controller: Recommended
  - Disk Type: Recommended 또는 NVMe/SCSI
  - Create a new virtual disk
  - Disk size: 60GB
  - Split virtual disk into multiple files
  - Customize Hardware 확인
  - ISO 연결
  - Power on
  - Desktop Experience 선택
  - Administrator 비밀번호 설정
  - VMware Tools 설치
  - ipconfig 확인
  - IIS 설치
  - localhost 접속
  - index.html 만들기
  - 필요하면 Bridged 전환

---

## 39. Side Channel Mitigations 안내창

### 사용자가 한 것
- VMware에서 Side Channel Mitigations 관련 안내창을 보여줬다.

### 질문
- “이게 뭐야”

### 답변 요약
- CPU 보안 취약점 계열을 막기 위한 완화 기능이 켜져 있어 성능이 약간 낮아질 수 있다는 안내라고 설명했다.
- 지금은 끄지 말고 OK를 누르라고 안내했다.

---

## 40. 설치 시간 질문

### 질문
- “설치 오래걸리나”

### 답변 요약
- 보통 10~30분 정도 걸린다고 설명했다.
- 재부팅 후 다시 `Press any key to boot from CD or DVD...`가 떠도 그때는 누르지 말라고 설명했다.

---

## 41. Windows 설치 오류

### 사용자가 한 것
- Windows 설치 중 다음 오류를 보여줬다.

```text
Windows cannot find the Microsoft Software License Terms.
Make sure the installation sources are valid and restart the installation.
```

### 질문
- “이게 뭐야?”

### 답변 요약
- ISO가 깨졌다기보다 VMware의 Easy Install 자동설치 기능이 Windows Server 설치와 충돌했을 가능성이 높다고 설명했다.
- 해결 방법으로 VM을 다시 만들되, ISO 선택 단계에서 `I will install the operating system later`를 선택하라고 안내했다.
- VM 생성 후 CD/DVD 설정에서 ISO를 수동 연결하는 방식을 추천했다.

---

## 42. VM 다시 만들기

### 사용자가 한 것
- VM 이름과 경로 설정 화면을 다시 보여줬다.
- 이름은 `Windows Server 2022 IIS Practice`
- 경로는 `C:\VMs\WindowsServer2022-IIS`

### 질문
- “이거 맞지?”

### 답변 요약
- 맞다고 답했다.
- 이름과 경로가 적절하다고 설명했다.
- 이후 설정은 이전처럼 다음으로 가면 된다고 안내했다.

```text
UEFI
CPU: 1 processor / 4 cores
RAM: 8192 MB
Network: NAT
Disk: 60GB
Split virtual disk into multiple files
```

---

## 43. 다시 만든 VM 이후 진행 안내

### 질문
- “이제 뭐해?”

### 답변 요약
- 다시 VM을 만드는 중이며, 핵심은 Easy Install을 피해서 수동 설치용 VM을 만드는 것이라고 설명했다.
- 단계별로 CPU, 메모리, NAT, I/O Controller, Disk Type, Disk Capacity, Disk File, Customize Hardware 확인, ISO 수동 연결, 부팅 과정을 안내했다.

---

## 44. CD/DVD 장치 연결 오류

### 사용자가 한 것
- 다음 메시지를 보여줬다.

```text
Cannot connect the virtual device sata0:1 because no corresponding device is available on the host.
Do you want to try to connect this virtual device every time you power on the virtual machine?
```

### 질문
- “이게 뭐야?”

### 답변 요약
- VM의 가상 CD/DVD 장치가 호스트 PC에서 연결할 대상을 못 찾았다는 뜻이라고 설명했다.
- 일단 `No`를 누르는 것이 낫다고 안내했다.
- 원인으로 다음을 설명했다.
  - CD/DVD가 물리 드라이브로 잡혀 있음
  - ISO 경로가 꼬였음
  - 클론된 VM을 켠 상태일 수 있음
- CD/DVD 설정에서 Windows Server ISO를 직접 연결하고 `Connect at power on`을 체크하라고 안내했다.

---

## 45. UEFI Boot Manager 화면

### 사용자가 한 것
- UEFI Boot Manager 화면을 보여줬다.
- 항목에는 다음이 있었다.
  - Boot normally
  - EFI VMware Virtual NVME Namespace
  - EFI VMware Virtual SATA CDROM Drive
  - EFI Network
  - EFI Internal Shell

### 질문
- “이게 뭐야?”

### 답변 요약
- 아직 설치된 OS가 없거나 ISO로 제대로 부팅하지 못해서 부팅 장치를 고르는 화면이라고 설명했다.
- `EFI VMware Virtual SATA CDROM Drive (1.0)`를 선택하라고 안내했다.

---

## 46. ISO 파일 확인

### 사용자가 한 것
- 다운로드 폴더에 있는 ISO 파일을 보여줬다.

```text
SERVER_EVAL_x64FRE_en-us.iso
```

- 용량은 4.70GB였다.

### 질문
- “이거 맞지 않아?”

### 답변 요약
- ISO 파일 자체는 맞아 보인다고 답했다.
- 문제는 ISO 파일이 아니라 VM의 CD/DVD 장치에 이 ISO가 제대로 연결되어 있는지라고 설명했다.
- 다시 CD/DVD 설정에서 해당 ISO를 연결하고 `Connected`, `Connect at power on`을 체크하라고 안내했다.

---

## 47. EFI Network Timeout

### 사용자가 한 것
- 다음 화면을 보여줬다.

```text
EFI Network...
Time out.
```

### 질문
- “이건 뭐야?”

### 답변 요약
- VM이 ISO/CD로 부팅하지 못하고 네트워크 부팅을 시도하다가 실패한 화면이라고 설명했다.
- 원인은 다음 중 하나라고 설명했다.
  - ISO가 CD/DVD에 연결 안 됨
  - `Press any key to boot from CD or DVD...` 타이밍을 놓침
- 다시 ISO 연결을 확인하고, Boot Manager에서 CDROM을 선택한 뒤, Press any key 메시지가 나오면 바로 키를 누르라고 안내했다.

---

## 48. Windows Server 설치 화면 진입

### 사용자가 한 것
- `Press any key`를 누르자 Windows Server 설치 화면에 들어왔다.
- 언어 화면을 보여줬다.
- 설정은 다음과 같았다.
  - Language to install: English (United States)
  - Time and currency format: Korean (Korea)
  - Keyboard or input method: Microsoft IME
  - Keyboard Type: Korean PC/AT 101-Key Compatible Keyboard (Type1)

### 질문
- “press key 하니까 되네. 뭐하면돼?”

### 답변 요약
- 정상적으로 Windows Server 설치 화면에 들어온 상태라고 설명했다.
- 현재 설정 그대로 Next를 누르면 된다고 안내했다.
- 이후 과정으로 다음을 안내했다.
  - Install now
  - Desktop Experience 붙은 에디션 선택
  - 약관 동의
  - Custom 설치 선택
  - Drive 0 Unallocated Space 선택
  - Next
- 재부팅 후 다시 `Press any key`가 떠도 누르지 말라고 설명했다.

---

## 49. Server Core 설치 완료

### 사용자가 한 것
- 설치 후 `SConfig` 화면을 보여줬다.
- 화면에는 다음 문구가 있었다.

```text
Welcome to Windows Server 2022 Standard Evaluation
```

- SConfig 메뉴가 보였다.
  - Domain/workgroup
  - Computer name
  - Add local administrator
  - Remote management
  - Update setting
  - Install updates
  - Remote desktop
  - Network settings
  - Date and time
  - Telemetry setting
  - Windows activation
  - Log off user
  - Restart server
  - Shut down server
  - Exit to command line (PowerShell)

### 질문
- “이건 뭐야?”

### 답변 요약
- Windows Server Core로 설치된 상태라고 설명했다.
- GUI가 있는 `Desktop Experience`가 아니라 명령어 중심 서버 버전으로 설치되었다고 설명했다.
- 설치가 실패한 것은 아니고, GUI 없는 서버로 설치된 것이라고 설명했다.
- 선택지로 다음을 설명했다.
  - 그대로 Server Core로 진행
  - 다시 설치해서 Desktop Experience 선택
- 처음 실습에는 Desktop Experience가 편하지만, Core도 실무 감각 측면에서는 의미가 있다고 설명했다.

---

## 50. Server Core가 실제 운영에 더 현실적인지 질문

### 질문
- “이거 네관사나 실제 서버 운영에는 이게 더 현실적이야?”

### 답변 요약
- 실제 서버 운영 관점에서는 Server Core가 더 현실적인 경우가 많다고 설명했다.
- 네트워크관리사 2급/입문 실습 관점에서는 GUI 있는 Desktop Experience가 더 좋다고 설명했다.
- Server Core의 장점으로 다음을 설명했다.
  - 공격 표면 감소
  - 자원 사용량 적음
  - PowerShell/원격 관리/자동화에 적합
- 네관사 관점에서는 Core냐 GUI냐보다 IP, 서브넷, 게이트웨이, DNS, DHCP, 포트, 프로토콜 이해가 더 중요하다고 설명했다.
- 추천은 다음과 같았다.
  - 처음 구조 이해: Desktop Experience
  - 실무 운영 감각: Server Core
- 현실적 최적안으로 GUI로 먼저 구조를 보고, 나중에 Core로 같은 작업을 반복하는 방식을 제안했다.

---

## 51. 현재 설치 완료 여부 확인

### 질문
- “그러면 하고싶을때 gui 쓰고 그냥 cli로 갈게. 일단 설치 끝난거지?”

### 답변 요약
- 설치는 끝난 상태라고 답했다.
- 현재 상태를 다음처럼 정리했다.

```text
Windows Server 2022 Standard Evaluation
Server Core 설치 완료
SConfig 초기 설정 화면 진입 완료
```

- 지금부터는 설치 단계가 아니라 서버 설정/운영 단계라고 설명했다.
- SConfig에서 할 수 있는 주요 작업을 설명했다.
- 다음으로 할 추천 순서를 안내했다.

```text
1. SConfig에서 PowerShell로 나가기
2. IP 확인
3. 인터넷 연결 확인
4. 컴퓨터 이름 변경
5. 재부팅
6. IIS 설치
7. localhost 테스트
8. 호스트 PC에서 접속 테스트
```

- 현재 화면에서 `15`를 입력하면 PowerShell로 나갈 수 있다고 안내했다.
- 이후 확인 명령어를 안내했다.

```powershell
ipconfig
ping 8.8.8.8
ping google.com
```

- IIS 설치 명령어를 안내했다.

```powershell
Install-WindowsFeature Web-Server
Get-WindowsFeature Web-Server
Invoke-WebRequest http://localhost
curl http://localhost
```

---

## 현재 최종 상태

현재까지 완료된 것:

```text
1. VMware Workstation Pro 설치 완료
2. Windows Server 2022 Evaluation ISO 다운로드 완료
3. VMware에서 Windows Server VM 생성
4. Easy Install 오류를 겪음
5. Easy Install을 피해서 수동 설치 방식으로 다시 진행
6. ISO/CD 부팅 문제를 겪고 해결
7. Windows Server 설치 화면 진입 성공
8. Windows Server 2022 Standard Evaluation 설치 완료
9. 현재 Server Core / SConfig 화면까지 진입 완료
```

현재 설치된 형태:

```text
Windows Server 2022 Standard Evaluation
Server Core
CLI 중심 환경
```

아직 하지 않은 것:

```text
1. PowerShell 진입
2. 네트워크 확인
3. 컴퓨터 이름 변경
4. IIS 설치
5. localhost 테스트
6. 호스트 PC에서 VM IIS 접속 테스트
7. 필요 시 Bridged 전환 테스트
```
