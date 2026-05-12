# Docker / WSL / 네트워크 질의 기록

작성 목적: 이 대화세션에서 도커, WSL, 리눅스 네트워크 실습 중 나온 질문과 핵심 답변을 나중에 빠르게 복습하기 위함.

---

## 1. Docker 네트워크 목록

명령어:

```bash
docker network ls
```

확인한 네트워크:

```text
bridge           Docker 기본 bridge 네트워크
host             호스트 네트워크 공유
none             네트워크 없음
docker_default   docker compose가 만든 bridge 네트워크
net-practice     직접 만든 실습용 bridge 네트워크
```

핵심:

```text
기본 네트워크: bridge, host, none
사용자 정의 네트워크: docker_default, net-practice
```

---

## 2. 컨테이너 내부에서 `ip addr`

명령어:

```bash
ip addr
```

예시 출력 핵심:

```text
lo
  127.0.0.1/8

eth0
  172.19.0.2/16
  brd 172.19.255.255
```

의미:

```text
lo = 컨테이너 자기 자신
eth0 = 컨테이너의 가상 랜카드
172.19.0.2 = 이 컨테이너가 Docker 네트워크 안에서 받은 IP
172.19.0.0/16 = 이 컨테이너가 속한 네트워크 대역
172.19.255.255 = 브로드캐스트 주소
```

중요한 관점:

```text
ip addr는 Docker 네트워크 전체를 보여주는 게 아니라,
현재 컨테이너 내부 Ubuntu 입장에서 보이는 IP와 인터페이스를 보여준다.
```

---

## 3. Docker 네트워크 전체 설정 확인

명령어:

```bash
docker network inspect net-practice
```

핵심 필드:

```text
Scope: local
  현재 Docker 엔진 내부에서만 쓰는 네트워크

Driver: bridge
  Docker bridge 네트워크

IPAM
  IP Address Management
  컨테이너에게 IP를 어떻게 나눠줄지 관리

IPAM.Driver: default
  Docker 기본 IP 할당 방식

Subnet: 172.18.0.0/16
  이 네트워크의 IP 대역

Gateway: 172.18.0.1
  컨테이너가 외부로 나갈 때 거치는 게이트웨이

Internal: false
  외부 통신 가능

Attachable: false
  Swarm에서 일반 컨테이너 수동 연결 허용 여부. 일반 bridge 실습에서는 중요도 낮음

Ingress: false
  Docker Swarm 외부 진입용 네트워크가 아님

ConfigFrom: ""
  다른 네트워크 설정을 가져오지 않음

ConfigOnly: false
  실제 통신 가능한 네트워크

Options
  드라이버 세부 옵션

Labels: {}
  사용자가 붙인 메타데이터 없음

Status
  현재 IP 사용 상태
```

---

## 4. `EnableIPv4/6`와 `Options.enable_ipv4/6`

```json
"EnableIPv4": true,
"EnableIPv6": false
```

상위 요약 필드에 가깝다.

```json
"Options": {
  "com.docker.network.enable_ipv4": "true",
  "com.docker.network.enable_ipv6": "false"
}
```

네트워크 드라이버에 저장된 세부 옵션이다.

정리:

```text
EnableIPv4/6 = 결과적으로 IPv4/IPv6가 켜졌는지
Options.enable_ipv4/6 = 드라이버 옵션으로 저장된 설정값
```

---

## 5. `IPsInUse`

예시:

```text
IPsInUse: 3
DynamicIPsAvailable: 65533
```

컨테이너가 없는데 3개가 사용 중으로 보이는 이유:

```text
172.18.0.0       네트워크 주소, 할당 불가
172.18.0.1       게이트웨이
172.18.255.255   브로드캐스트 주소, 할당 불가
```

정확히는 Docker IPAM이 컨테이너에게 줄 수 없는 예약 주소까지 사용 중처럼 계산할 수 있다.

---

## 6. 네트워크 주소 `172.18.0.0`

질문 핵심:

```text
172.18.0.0은 실제로 어디에 쓰이는가?
왜 호스트 IP로 못 쓰는가?
```

정리:

```text
172.18.0.0은 특정 장비 주소가 아니라 네트워크 대역을 대표하는 주소다.
```

예:

```text
172.18.0.0/16
= 172.18.0.0 ~ 172.18.255.255 전체 대역
```

`/16` 기준:

```text
172.18 | 0.0
네트워크 부분 | 호스트 부분
```

규칙:

```text
호스트 부분이 전부 0 → 네트워크 주소
호스트 부분이 전부 1 → 브로드캐스트 주소
```

그래서:

```text
172.18.0.0
= 호스트 부분이 전부 0
= 172.18 네트워크 자체를 의미
= 개별 컨테이너/장비에 할당하지 않음
```

---

## 7. `172.18.0.0`과 `172.18.0.0/16` 차이

```text
172.18.0.0
= 주소값 하나처럼 보이지만, /16 맥락에서는 네트워크 주소

172.18.0.0/16
= 172.18.0.0부터 시작하는 /16 네트워크 전체
= 172.18.0.0 ~ 172.18.255.255
```

네트워크를 말할 때는 보통 `/16`까지 붙여야 정확하다.

---

## 8. 호스트 부분이 0인 주소를 쓰는 경우

일반적인 목적지 IP로는 거의 안 쓴다.

주요 사용:

```text
1. 네트워크 대역 표현
   172.18.0.0/16

2. 라우팅 테이블
   172.18.0.0/16 dev eth0

3. 특수 주소 0.0.0.0
   아직 IP 없음
   모든 인터페이스에서 수신
   기본 경로
```

예:

```text
0.0.0.0:8080
= 이 장비의 모든 인터페이스에서 8080 포트를 받음
```

---

## 9. 컨테이너 내부 `ip route`

예시:

```bash
ip route
```

출력:

```text
default via 172.19.0.1 dev eth0
172.19.0.0/16 dev eth0 proto kernel scope link src 172.19.0.2
```

해석:

```text
172.19.0.0/16 dev eth0 src 172.19.0.2
= 목적지가 172.19.x.x면 eth0로 직접 보냄
= 출발지 IP는 172.19.0.2

default via 172.19.0.1 dev eth0
= 그 외 목적지는 eth0로 나가서 172.19.0.1 게이트웨이를 거침
```

라우팅은 아래서부터 순서대로 검사하는 게 아니라:

```text
가장 구체적으로 맞는 경로가 우선
```

---

## 10. `via`, `dev`, `proto kernel`, `scope link`, `src`

```text
via
  경유지. 게이트웨이를 뜻함.
  예: via 172.19.0.1

dev
  device의 약자.
  어떤 네트워크 인터페이스로 보낼지.
  예: dev eth0

proto kernel
  커널이 자동으로 만든 라우트

scope link
  같은 네트워크 링크 안에서 직접 도달 가능

src
  이 경로를 사용할 때 출발지로 사용할 IP
```

예:

```text
172.19.0.0/16 dev eth0 proto kernel scope link src 172.19.0.2
```

뜻:

```text
172.19.x.x 대역은 eth0로 직접 보내고,
출발지 IP는 172.19.0.2를 사용한다.
```

---

## 11. Docker DNS `127.0.0.11`

컨테이너 안에서는 Docker 내장 DNS에 직접 질의 가능.

확인:

```bash
cat /etc/resolv.conf
```

예:

```text
nameserver 127.0.0.11
```

질의:

```bash
apt install -y dnsutils
dig @127.0.0.11 box2
nslookup box2 127.0.0.11
```

단, PowerShell 호스트에서는 보통 불가.

이유:

```text
PowerShell의 127.0.0.11 = Windows 자기 자신의 루프백
컨테이너의 127.0.0.11 = 컨테이너 네임스페이스 내부 Docker DNS
```

---

## 12. Docker가 `127.0.0.11`을 쓰는 원리

핵심:

```text
네트워크 네임스페이스가 다르기 때문
```

```text
Windows 호스트 namespace
  127.0.0.11 = Windows 자기 자신

컨테이너 namespace
  127.0.0.11 = 컨테이너 내부 Docker DNS
```

Docker가 호스트 전체의 `127.0.0.11`을 점유한 게 아니다.

컨테이너마다 분리된 네트워크 공간 안에 `127.0.0.11`을 둔 것이다.

---

## 13. 호스트 DNS 확인

Windows PowerShell:

```powershell
ipconfig /all
```

또는:

```powershell
Get-DnsClientServerAddress
```

WSL Ubuntu:

```bash
cat /etc/resolv.conf
```

Docker 컨테이너:

```bash
cat /etc/resolv.conf
```

---

## 14. Windows `ipconfig /all` 핵심 해석

실제 이더넷 어댑터:

```text
IPv4 주소: 192.168.45.96
서브넷 마스크: 255.255.255.0
기본 게이트웨이: 192.168.45.1
DHCP 서버: 192.168.45.1
DNS 서버: 210.220.163.82, 219.250.36.130
MAC 주소: B0-82-E2-02-4C-AE
```

의미:

```text
192.168.45.96 = Windows PC의 LAN 사설 IP
192.168.45.1 = 공유기/라우터
DNS 서버 = 도메인을 IP로 바꿔주는 서버
MAC 주소 = 물리 랜카드 주소
```

WSL 가상 어댑터:

```text
vEthernet (WSL)
IPv4 주소: 172.27.208.1
서브넷 마스크: 255.255.240.0
```

의미:

```text
Windows가 WSL과 통신하기 위해 만든 가상 네트워크 어댑터
```

---

## 15. `자동 구성 사용`, 링크-로컬 IPv6

```text
자동 구성 사용: 예
```

Windows가 DHCP/IPv6 자동 구성 등을 사용해 네트워크 설정을 자동으로 잡을 수 있다는 뜻.

```text
링크-로컬 IPv6 주소: fe80::...%13
```

의미:

```text
fe80::/10 = 같은 링크 안에서만 쓰는 IPv6 주소
%13 = 13번 네트워크 인터페이스
```

인터넷 전체용 주소가 아니라 같은 LAN/링크 안에서 쓰는 주소다.

---

## 16. WSL에서 외부로 나가는 경로

일반적인 WSL NAT 모드 기준:

```text
WSL Ubuntu
→ Windows의 WSL 가상 어댑터
→ Windows 네트워크 스택
→ 실제 이더넷 어댑터
→ 공유기/라우터 192.168.45.1
→ 인터넷
```

네 환경 기준:

```text
WSL
→ vEthernet (WSL) 172.27.208.1
→ Windows
→ Realtek 이더넷 어댑터
   IP: 192.168.45.96
   MAC: B0-82-E2-02-4C-AE
→ 게이트웨이 192.168.45.1
→ 인터넷
```

단, WSL mirrored networking, VPN, 프록시 설정에 따라 달라질 수 있다.

---

## 17. WSL과 Windows 파일시스템

Windows:

```text
C:\
├─ Windows\
├─ Program Files\
├─ Users\
│  └─ hurwa\
│     ├─ Desktop\
│     ├─ Documents\
│     ├─ Downloads\
│     └─ AppData\
└─ ...
```

WSL Ubuntu:

```text
/
├─ home/
│  └─ user/
├─ etc/
├─ usr/
├─ var/
├─ root/
└─ mnt/
   ├─ c/
   └─ d/
```

중요 경로:

```text
/home/user
= WSL Ubuntu 내부 사용자 홈

/mnt/c/Users/hurwa
= WSL에서 접근한 Windows 사용자 폴더

C:\Users\hurwa
= Windows 원래 사용자 폴더

\\wsl.localhost\Ubuntu\home\user
= Windows에서 접근한 WSL Ubuntu 폴더
```

---

## 18. WSL은 Windows 내부를 Ubuntu처럼 보여주는 것인가?

아니다.

정확히는:

```text
Windows 안에 별도의 Ubuntu 리눅스 환경을 설치하고,
Windows와 Ubuntu가 서로 파일시스템에 접근할 수 있게 해주는 것
```

구조:

```text
Windows
├─ C:\Windows
├─ C:\Users\hurwa
└─ WSL
   └─ Ubuntu
      ├─ /home/user
      ├─ /etc
      ├─ /usr
      └─ /var
```

---

## 19. WSL, Docker, VMware 차이

```text
WSL
  Windows 안의 Linux 개발환경
  가볍고 빠름

Docker
  애플리케이션/서버 실행용 격리 환경
  컨테이너 중심

VMware
  컴퓨터 한 대를 통째로 가상화
  OS 전체 실습에 적합
```

추천 흐름:

```text
기본 작업 환경: WSL Ubuntu
서버/DB 실습: Docker
OS 깊은 실험: VMware는 나중에
```

---

## 20. Ubuntu 컨테이너를 `sleep infinity`로 실행하는 이유

Ubuntu 이미지는 기본적으로 계속 실행되는 프로그램이 없다.

```bash
docker run ubuntu
```

하면 할 일이 없어서 바로 종료될 수 있다.

실습용 실행:

```bash
docker run -dit --name u1 --network net-practice ubuntu sleep infinity
docker exec -it u1 bash
```

의미:

```text
sleep infinity = 컨테이너를 계속 살아있게 하는 메인 프로세스
bash = exec로 들어가서 쓰는 임시 셸
```

---

## 21. `docker run -dit ubuntu bash`가 애매한 이유

```bash
docker run -d -i -t ubuntu bash
```

의미:

```text
-d = 백그라운드
-i = STDIN 열어둠
-t = TTY 할당
bash = 컨테이너 메인 프로세스
```

문제:

```text
bash는 입력을 받는 대화형 셸인데,
-d는 백그라운드 실행이다.
```

`-dit`이면 살아있을 수는 있지만, `bash`가 메인 프로세스라서 attach 후 exit하면 컨테이너가 종료될 수 있다.

실습용으로는 역할이 분리된 방식이 더 명확하다.

```bash
docker run -dit --name u1 ubuntu sleep infinity
docker exec -it u1 bash
```

---

## 22. `docker container ls -al`

```bash
docker container ls -al
```

옵션:

```text
-a = all
  실행 중 + 종료된 컨테이너 모두 보기

-l = latest
  가장 최근 생성된 컨테이너 1개 보기
```

보통은 따로 쓴다.

```bash
docker container ls
docker container ls -a
docker container ls -l
```

---

## 23. `ls -al`

```bash
ls -al
```

의미:

```text
ls = list
-a = all, 숨김파일 포함
-l = long, 자세히 보기
```

출력 컬럼:

```text
권한  링크수  소유자  그룹  크기  수정시간  이름
```

예:

```text
-rw-r--r-- 1 user user 3771 May 11 17:50 .bashrc
```

해석:

```text
일반 파일
소유자 읽기/쓰기 가능
그룹/기타 읽기 가능
소유자 user
그룹 user
크기 3771바이트
이름 .bashrc
```

첫 글자:

```text
d = 디렉터리
- = 일반 파일
l = 심볼릭 링크
```

---

## 24. `l`은 인터넷 링크인가?

아니다.

```text
l = symbolic link
```

파일시스템 안의 바로가기다.

예:

```text
python -> /usr/bin/python3
```

마운트와 차이:

```text
심볼릭 링크 = 특정 파일/폴더를 가리키는 바로가기
마운트 = 다른 디스크/파일시스템을 특정 폴더에 연결
```

예:

```text
/mnt/c
= Windows C드라이브가 WSL에 마운트된 것
```

---

## 25. `.aws`, `.azure`

```text
~/.aws
~/.azure
```

보통 클라우드 CLI 설정 폴더다.

```text
.aws
  AWS CLI 설정/인증 정보

.azure
  Azure CLI 설정/로그인 캐시
```

안이 비어있으면 보통 문제 없음.

가능한 생성 이유:

```text
AWS CLI/Azure CLI 설치
VSCode 확장 프로그램
Docker/SDK/개발도구 설치 과정
```

비어있으면 삭제해도 대체로 문제 없지만, 필요 시 다시 생성될 수 있다.

---

## 26. WSL과 Windows는 같은 Docker Engine을 쓰는가?

Docker Desktop 사용 시 보통 같다.

```text
Windows PowerShell docker 명령어
→ Docker Desktop Engine

WSL Ubuntu docker 명령어
→ Docker Desktop Engine
```

확인:

```bash
docker context ls
```

Windows와 WSL에서 `docker container ls` 결과가 같으면 같은 Docker Engine을 보고 있는 것이다.

단, WSL 내부에 Docker Engine을 따로 설치했다면 별도 엔진일 수 있다.

---

## 27. `tzdata` 시간대 설정

설치 중 나온 질문:

```text
Geographic area: 5 → Asia
Time zone: 67 → Seoul
```

의미:

```text
Ubuntu 내부 시간대를 Asia/Seoul로 설정
```

파이썬 자체 설정이라기보다, `python3`, `python3-pip`, 또는 다른 패키지 설치 중 의존 패키지인 `tzdata`가 시간대를 물어본 것일 가능성이 높다.

비대화형 설치 예:

```bash
DEBIAN_FRONTEND=noninteractive TZ=Asia/Seoul apt install -y tzdata python3 python3-pip
```

---

# 전체 흐름 요약

```text
Windows 실제 네트워크
  192.168.45.96
  게이트웨이 192.168.45.1
  DNS 210.220.163.82, 219.250.36.130

WSL 가상 네트워크
  Windows 쪽 vEthernet: 172.27.208.1

Docker 사용자 정의 네트워크
  net-practice
  예: 172.18.0.0/16
  Gateway: 172.18.0.1

Docker 컨테이너
  예: 172.18.0.2
  DNS: 127.0.0.11
  루프백: 127.0.0.1
```

핵심 관점:

```text
Windows, WSL, Docker 컨테이너는 각각 다른 네트워크 관점을 가진다.
같은 127.0.0.1이라도 어느 namespace에서 보느냐에 따라 가리키는 대상이 다르다.
```
