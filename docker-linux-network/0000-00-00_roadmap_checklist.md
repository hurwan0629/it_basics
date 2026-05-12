# Docker / Linux / Network 학습 로드맵 체크리스트

작성 목적: 지금까지 진행한 Docker, WSL, Linux 네트워크 실습을 체크리스트로 정리하고, 앞으로 어떤 순서로 확장할지 기준을 잡기 위함.

---

## 0. 전체 방향

```text
목표:
단순히 명령어를 따라 치는 것이 아니라,
서버 / 네트워크 / 배포 구조를 직접 이해하고 진단할 수 있는 능력을 만드는 것.
```

최종적으로 얻고자 하는 것:

```text
1. localhost, IP, 포트, DNS, 라우팅을 구분할 수 있다.
2. Docker 컨테이너 간 통신 구조를 이해한다.
3. Spring Boot / PostgreSQL / Next.js 같은 실제 웹 개발 구조와 연결한다.
4. 서버가 안 될 때 어디가 문제인지 직접 좁혀갈 수 있다.
5. Docker Compose, 웹서버, Swarm, Kubernetes로 자연스럽게 확장한다.
```

---

# 1. 환경 준비

## 1-1. WSL2 / Ubuntu 준비

- [x] Windows에서 WSL2 사용
- [x] Ubuntu 터미널 사용
- [x] WSL2 환경에서 Docker 명령어 실행 준비

### 이유

Docker와 Linux 네트워크 실습을 Windows 안에서 안정적으로 하기 위해서.

### 얻고자 하는 것

```text
Windows 안에서 Linux 개발환경을 쓰는 구조 이해
WSL과 Windows가 같은 시스템이 아니라 서로 연결된 별도 환경이라는 감각
```

---

## 1-2. Docker Desktop 연동

- [x] Docker Desktop 사용
- [x] Docker WSL Integration 사용
- [x] WSL Ubuntu에서 Docker 명령어 실행
- [x] Windows와 WSL이 같은 Docker Engine을 볼 수 있다는 점 확인

### 이유

실무에서 Docker Desktop + WSL 조합을 많이 사용하고, 이후 Docker Compose / Spring / DB 실습과 직접 연결되기 때문.

### 얻고자 하는 것

```text
Docker Client와 Docker Engine의 관계 이해
Windows PowerShell과 WSL Ubuntu에서 같은 컨테이너 목록이 보일 수 있는 이유 이해
```

---

## 1-3. 네트워크 도구 설치

- [x] curl 설치 또는 사용
- [x] iproute2 설치 또는 사용
- [x] ping 도구 설치 또는 사용
- [x] dnsutils 설치 또는 사용
- [x] netcat-openbsd 설치 또는 사용
- [ ] tcpdump 본격 실습

### 이유

네트워크 문제는 하나의 명령어로만 확인할 수 없고, 계층별로 나눠서 확인해야 하기 때문.

### 얻고자 하는 것

```text
ping  = IP 도달성 확인
nc    = TCP 포트 연결 확인
curl  = HTTP 응답 확인
ip    = 인터페이스 / 라우팅 확인
dig   = DNS 확인
tcpdump = 실제 패킷 관찰
```

---

# 2. Docker 기본 네트워크 이해

## 2-1. Docker 네트워크 목록 확인

- [x] docker network ls 실행
- [x] 기본 bridge 네트워크 확인
- [x] host 네트워크 확인
- [x] none 네트워크 확인
- [x] 사용자 정의 네트워크 확인

### 이유

Docker 컨테이너가 아무렇게나 통신하는 것이 아니라, 특정 네트워크에 소속되어 통신한다는 점을 이해하기 위해서.

### 얻고자 하는 것

```text
기본 네트워크와 사용자 정의 네트워크 구분
Docker 네트워크가 컨테이너 통신의 단위라는 이해
```

---

## 2-2. 사용자 정의 bridge 네트워크 생성

- [x] net-practice 네트워크 생성
- [x] docker network inspect로 상세 설정 확인
- [x] Subnet 확인
- [x] Gateway 확인
- [x] Containers 항목 확인

### 이유

사용자 정의 bridge 네트워크에서는 컨테이너 이름 기반 DNS가 동작하므로, 실무적인 Docker 통신 구조를 배우기 좋다.

### 얻고자 하는 것

```text
Docker 네트워크 대역 이해
172.18.0.0/16 같은 Subnet 의미 이해
172.18.0.1 Gateway 의미 이해
컨테이너가 네트워크에 붙으면 IP를 받는다는 이해
```

---

## 2-3. 네트워크 주소 / 브로드캐스트 / 게이트웨이 이해

- [x] 172.18.0.0 의미 정리
- [x] 172.18.0.0/16 의미 정리
- [x] 172.18.0.1 Gateway 의미 정리
- [x] 172.18.255.255 Broadcast 의미 정리
- [x] IPsInUse가 컨테이너 수와 다를 수 있는 이유 정리

### 이유

IP 주소를 단순 숫자로 보는 것이 아니라, 네트워크 대역과 호스트 주소로 나눠서 보기 위해서.

### 얻고자 하는 것

```text
네트워크 주소는 장비 주소가 아니라 대역을 대표한다는 이해
호스트 부분이 전부 0이면 네트워크 주소
호스트 부분이 전부 1이면 브로드캐스트 주소
```

---

# 3. 컨테이너 내부 네트워크 확인

## 3-1. 컨테이너 ip addr 확인

- [x] 컨테이너 내부에서 ip addr 실행
- [x] lo 확인
- [x] eth0 확인
- [x] 컨테이너 IP 확인
- [x] 127.0.0.1과 컨테이너 IP 차이 이해

### 이유

같은 `ip addr`라도 Windows, WSL, Docker 컨테이너에서 보는 네트워크 관점이 다르기 때문.

### 얻고자 하는 것

```text
lo = 자기 자신
eth0 = 컨테이너의 가상 네트워크 인터페이스
컨테이너 IP = Docker 네트워크 안에서 쓰는 IP
```

---

## 3-2. 컨테이너 ip route 확인

- [x] ip route 실행
- [x] default via 의미 정리
- [x] dev 의미 정리
- [x] proto kernel 의미 정리
- [x] scope link 의미 정리
- [x] src 의미 정리

### 이유

IP가 어디로 나가는지 이해하려면 인터페이스뿐 아니라 라우팅 테이블을 봐야 하기 때문.

### 얻고자 하는 것

```text
같은 대역이면 직접 eth0로 보냄
다른 대역이면 default gateway로 보냄
라우팅은 가장 구체적으로 맞는 경로가 우선됨
```

---

# 4. Docker 컨테이너 간 통신

## 4-1. linux1 / linux2 컨테이너 실행

- [x] linux1 컨테이너 실행
- [x] linux2 컨테이너 실행
- [x] 둘 다 net-practice 네트워크에 연결
- [x] sleep infinity로 컨테이너 유지

### 이유

네트워크 실습을 위해서는 서로 통신할 수 있는 두 개 이상의 독립된 환경이 필요하기 때문.

### 얻고자 하는 것

```text
컨테이너 하나 = 독립된 작은 Linux 환경
같은 Docker network 안의 컨테이너끼리는 통신 가능
sleep infinity는 실습용 컨테이너를 계속 살려두는 메인 프로세스
```

---

## 4-2. 컨테이너 이름 기반 통신

- [x] linux1에서 linux2로 ping
- [x] linux2 이름이 IP로 해석되는 것 확인
- [x] 같은 Docker network 안에서 이름으로 통신 가능하다는 점 이해

### 이유

실제 Docker Compose에서 Spring이 PostgreSQL에 접속할 때 `localhost`가 아니라 `postgres` 같은 서비스 이름을 쓰기 때문.

### 얻고자 하는 것

```text
Docker 사용자 정의 네트워크 안에서는 컨테이너 이름이 DNS 이름처럼 동작한다.
IP를 직접 외우지 않아도 이름으로 접근할 수 있다.
```

---

## 4-3. Docker 내부 DNS 확인

- [x] /etc/resolv.conf 확인
- [x] nameserver 127.0.0.11 확인
- [x] Docker 내부 DNS 개념 정리
- [x] Windows의 127.0.0.11과 컨테이너의 127.0.0.11이 다르다는 점 이해

### 이유

`127.0.0.1`, `127.0.0.11`, `localhost` 같은 주소는 어느 네트워크 namespace에서 보느냐에 따라 의미가 달라지기 때문.

### 얻고자 하는 것

```text
컨테이너마다 독립된 네트워크 namespace가 있다.
컨테이너 내부의 127.0.0.11은 Docker가 제공하는 DNS다.
호스트의 127.0.0.11과 같은 것이 아니다.
```

---

# 5. HTTP 서버 실습

## 5-1. linux2에서 Python HTTP 서버 실행

- [x] linux2 컨테이너에 python3 설치
- [x] index.html 생성
- [x] python3 -m http.server 8000 실행

### 이유

네트워크 통신을 단순 ping이 아니라 실제 HTTP 요청 / 응답 흐름으로 확인하기 위해서.

### 얻고자 하는 것

```text
서버는 특정 포트를 열고 요청을 기다린다.
HTTP 요청은 IP 도달성보다 한 단계 위의 애플리케이션 통신이다.
```

---

## 5-2. linux1에서 linux2 HTTP 요청

- [x] linux1에서 curl http://linux2:8000 실행
- [x] hello from linux2 응답 확인
- [x] 컨테이너 이름 DNS + HTTP 서버 흐름 이해

### 이유

브라우저 / 프론트엔드 / 백엔드 / DB 간 통신도 결국 이런 요청과 응답 구조이기 때문.

### 얻고자 하는 것

```text
클라이언트 컨테이너 → 서버 컨테이너
이름 해석 → IP 확인 → 포트 연결 → HTTP 응답
```

---

## 5-3. ping / nc / curl 차이

- [x] ping 의미 정리
- [x] nc 의미 정리
- [x] curl 의미 정리
- [ ] 장애 상황을 만들어서 ping 성공 / nc 실패 / curl 실패 구분 실습

### 이유

서버가 안 될 때 “안 된다”로 끝내지 않고 어느 계층에서 막혔는지 구분하기 위해서.

### 얻고자 하는 것

```text
ping 성공 = 네트워크 레벨 도달 가능
nc 성공 = TCP 포트 열림
curl 성공 = HTTP 애플리케이션 응답 정상
```

---

# 6. localhost 기준 이해

## 6-1. Windows / WSL / 컨테이너 localhost 구분

- [x] Windows의 localhost 의미 정리
- [x] WSL의 localhost 의미 정리
- [x] 컨테이너 내부 localhost 의미 정리
- [x] Spring 컨테이너에서 localhost가 PostgreSQL이 아니라 Spring 자기 자신이라는 점 정리

### 이유

Docker와 웹 개발에서 가장 자주 막히는 원인이 `localhost` 오해이기 때문.

### 얻고자 하는 것

```text
localhost는 항상 지금 명령을 실행하는 자기 자신이다.
컨테이너끼리 통신할 때는 localhost가 아니라 컨테이너 이름을 써야 한다.
```

---

# 7. 포트 매핑 실습

## 7-1. 호스트 포트와 컨테이너 포트 연결

- [x] web1 컨테이너 실행
- [x] -p 8080:8000 설정
- [x] web1 내부에서 Python HTTP 서버 실행
- [x] WSL에서 curl http://localhost:8080 확인
- [x] Windows PowerShell에서 curl http://localhost:8080 확인
- [x] 브라우저에서 http://localhost:8080 확인

### 이유

컨테이너 내부 서버를 내 PC 브라우저나 WSL에서 접근하려면 포트 매핑이 필요하기 때문.

### 얻고자 하는 것

```text
호스트 포트:컨테이너 포트 구조 이해
localhost:8080이 컨테이너의 localhost가 아니라 내 PC 쪽 입구라는 점 이해
```

---

## 7-2. 내부 접근과 외부 접근 비교

- [x] 호스트에서 localhost:8080으로 접근
- [x] web1 컨테이너 내부에서 localhost:8000으로 접근
- [x] client1 컨테이너에서 web1:8000으로 접근
- [x] 세 접근 방식의 차이 정리

### 이유

같은 서버라도 출발지가 어디냐에 따라 주소가 달라지기 때문.

### 얻고자 하는 것

```text
내 PC → 컨테이너: localhost:8080
컨테이너 → 컨테이너: web1:8000
컨테이너 자기 자신 → 자기 자신: localhost:8000
```

---

# 8. Linux shell script 프로젝트

## 8-1. network-check.sh

- [x] 인자로 host와 port 받기
- [x] 현재 IP 출력
- [x] 라우팅 테이블 출력
- [x] DNS 조회
- [x] ping 테스트
- [x] nc 포트 테스트
- [x] curl HTTP 테스트

### 이유

지금까지 배운 네트워크 명령어를 하나의 진단 도구로 묶기 위해서.

### 얻고자 하는 것

```text
서버가 안 될 때 직접 원인을 좁히는 능력
단순 명령어 암기보다 실전 진단 흐름 습득
```

---

## 8-2. server-health.sh

- [ ] 특정 포트가 열려 있는지 확인
- [ ] 특정 프로세스가 떠 있는지 확인
- [ ] HTTP 응답 코드 확인
- [ ] 로그 마지막 줄 확인

### 이유

실제 서버 운영에서는 실행 여부, 포트, 응답, 로그를 함께 봐야 하기 때문.

### 얻고자 하는 것

```text
서버 상태 점검 루틴 만들기
운영 관점에서 문제를 확인하는 습관 만들기
```

---

## 8-3. .bat / PowerShell 간단 자동화

- [ ] Windows에서 curl 테스트용 bat 만들기
- [ ] Docker 상태 확인 bat 만들기
- [ ] 필요 시 PowerShell 스크립트로 확장

### 이유

Windows 환경에서도 반복 작업을 자동화할 수 있어야 하기 때문.

### 얻고자 하는 것

```text
Linux .sh와 Windows .bat / PowerShell의 역할 차이 이해
개발환경별 자동화 감각 확보
```

---

# 9. Docker Compose 실습

## 9-1. PostgreSQL 컨테이너 연결

- [ ] postgres 컨테이너 실행
- [ ] 환경변수로 DB 이름 / 유저 / 비밀번호 설정
- [ ] 볼륨으로 데이터 유지
- [ ] 다른 컨테이너에서 postgres:5432로 접속

### 이유

Spring Boot와 DB 연결 실습의 핵심이기 때문.

### 얻고자 하는 것

```text
DB 컨테이너 실행 방식 이해
컨테이너 이름으로 DB에 접속하는 구조 이해
볼륨이 필요한 이유 이해
```

---

## 9-2. Spring Boot + PostgreSQL Compose

- [ ] Spring Boot Dockerfile 작성
- [ ] compose.yml 작성
- [ ] spring 서비스 작성
- [ ] postgres 서비스 작성
- [ ] Spring에서 jdbc:postgresql://postgres:5432/testdb로 연결
- [ ] DB 연결 확인 API 작성

### 이유

현재 웹 백엔드 학습과 Docker 네트워크 개념을 직접 연결하기 위해서.

### 얻고자 하는 것

```text
실무형 개발환경 구성 경험
DB_HOST를 localhost가 아니라 서비스 이름으로 쓰는 이유 체득
```

---

# 10. Apache / IIS 웹서버 실습

## 10-1. Apache 기본 실습

- [ ] Apache 컨테이너 또는 Linux Apache 설치
- [ ] 정적 HTML 제공
- [ ] 포트 변경
- [ ] access log / error log 확인
- [ ] Apache reverse proxy 설정

### 이유

웹서버가 단순히 서버 프로그램이 아니라, 요청을 받아 정적 파일을 제공하거나 백엔드로 넘기는 앞단 역할을 하기 때문.

### 얻고자 하는 것

```text
웹서버와 백엔드 서버의 차이 이해
reverse proxy 개념 이해
로그 기반 문제 확인 경험
```

---

## 10-2. IIS 짧은 체험

- [ ] Windows 기능에서 IIS 활성화
- [ ] 기본 웹사이트 실행
- [ ] 정적 HTML 배포
- [ ] 포트 바인딩 확인
- [ ] IIS 로그 위치 확인

### 이유

Windows Server 계열 웹서버 환경도 한 번 경험해두면 회사 내부망, 레거시, ASP.NET 환경을 이해하는 데 도움이 되기 때문.

### 얻고자 하는 것

```text
Linux 웹서버와 Windows 웹서버의 차이 감각
IIS를 깊게 알기보다는 써본 경험 확보
```

---

# 11. Docker Swarm 맛보기

## 11-1. Swarm 기본

- [ ] docker swarm init
- [ ] docker service create
- [ ] docker service ls
- [ ] docker service ps
- [ ] docker service scale
- [ ] rolling update 실습

### 이유

컨테이너 하나를 실행하는 것과 여러 개의 컨테이너를 서비스 단위로 운영하는 것은 다르기 때문.

### 얻고자 하는 것

```text
서비스
레플리카
스케일링
롤링 업데이트
오케스트레이션 기본 감각
```

---

## 11-2. Swarm은 깊게 하지 않기

- [ ] overlay network 개념 확인
- [ ] manager / worker 개념 확인
- [ ] Kubernetes로 넘어가기 전 개념 맛보기로 마무리

### 이유

현재 실무 흐름에서는 Kubernetes가 더 중요하고, Swarm은 개념 이해용으로 충분하기 때문.

### 얻고자 하는 것

```text
Kubernetes에 들어가기 전 오케스트레이션이 무엇인지 가볍게 이해
```

---

# 12. Kubernetes 입문

## 12-1. 로컬 Kubernetes 환경

- [ ] Docker Desktop Kubernetes 또는 minikube / kind 선택
- [ ] kubectl 설치 및 확인
- [ ] cluster 상태 확인

### 이유

현대 배포 환경에서 Kubernetes 기본 개념은 점점 중요해지고 있기 때문.

### 얻고자 하는 것

```text
컨테이너 실행 단위가 Docker container에서 Pod / Deployment로 확장되는 흐름 이해
```

---

## 12-2. Kubernetes 기본 리소스

- [ ] Pod
- [ ] Deployment
- [ ] Service
- [ ] ConfigMap
- [ ] Secret
- [ ] Volume
- [ ] Ingress
- [ ] Rolling Update

### 이유

Kubernetes를 실무적으로 읽고 이해하려면 기본 리소스의 역할을 알아야 하기 때문.

### 얻고자 하는 것

```text
Pod = 실행 단위
Deployment = 배포 관리
Service = 내부 접근 경로
Ingress = 외부 접근 경로
ConfigMap / Secret = 설정 분리
Volume = 데이터 유지
```

---

# 13. 최종 미니 프로젝트

## 13-1. Mini Infra Lab

- [ ] Spring Boot API
- [ ] PostgreSQL
- [ ] Apache 또는 Nginx reverse proxy
- [ ] Docker Compose
- [ ] start.sh
- [ ] stop.sh
- [ ] health-check.sh
- [ ] logs.sh
- [ ] README.md 작성

### 이유

개별 실습을 하나의 구조로 묶어야 실제 포트폴리오나 학습 기록으로 남기 좋기 때문.

### 얻고자 하는 것

```text
서버 구조 설계 경험
실행 / 중지 / 점검 자동화 경험
Docker 네트워크와 웹서버 구조를 설명할 수 있는 능력
```

---

# 14. 추천 진행 순서 요약

```text
1. 포트 매핑 실습 마무리
2. ping / nc / curl 장애 구분 실습
3. network-check.sh 만들기
4. server-health.sh 만들기
5. PostgreSQL 컨테이너 연결
6. Spring Boot + PostgreSQL Docker Compose
7. Apache reverse proxy
8. IIS 짧게 체험
9. Docker Swarm 맛보기
10. Kubernetes 입문
11. Mini Infra Lab 완성
```

---

# 15. 현재 체크 상태 요약

## 완료한 것

```text
WSL2 / Ubuntu / Docker Desktop 기반 환경 준비
Docker 네트워크 목록 확인
사용자 정의 bridge 네트워크 생성
컨테이너 내부 ip addr 확인
컨테이너 내부 ip route 확인
Docker DNS 127.0.0.11 이해
linux1 / linux2 컨테이너 통신
컨테이너 이름 기반 ping
linux2 Python HTTP 서버 실행
linux1에서 linux2로 curl 요청
localhost 기준 정리
Windows / WSL / Docker 네트워크 차이 정리
```

## 다음에 할 것

```text
포트 매핑 실습
호스트 포트와 컨테이너 포트 구분
컨테이너 내부 접근 / 외부 접근 비교
network-check.sh 작성
```

---

# 16. 한 줄 기준

```text
앞으로의 공부는 “명령어를 아는 것”보다
“서버가 안 될 때 어디서 막혔는지 직접 확인할 수 있는 것”을 목표로 한다.
```
