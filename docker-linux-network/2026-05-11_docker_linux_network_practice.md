# Docker / Linux 네트워크 실습 정리

작성일: 2026-05-11  
목적: Docker와 Linux를 이용해 네트워크 개념을 직접 실습하기 위한 환경 준비 및 첫 네트워크 실습 정리

---

## 1. 오늘의 큰 목표

Docker와 Linux 환경을 이용해서 네트워크를 직접 연습할 수 있는 기반을 만들었다.

핵심 목표는 다음과 같다.

```text
Windows
  └─ WSL2
      └─ Ubuntu
          └─ Docker 명령어 사용
Docker Desktop
  └─ 실제 Docker 엔진 관리
```

즉, Docker Desktop을 백그라운드 Docker 엔진으로 사용하고, Ubuntu/WSL 터미널에서 Docker 명령어를 실행하는 구조다.

---

## 2. 왜 Docker로 네트워크를 연습하는가?

Docker는 네트워크 실습에 적합하다.

이유는 다음과 같다.

| 주제 | Docker로 연습 가능 여부 | 예시 |
|---|---:|---|
| IP 주소 | 가능 | 컨테이너 IP 확인 |
| 포트 | 가능 | `8080:8080`, `5432:5432` |
| DNS | 가능 | 컨테이너 이름으로 접속 |
| Bridge Network | 가능 | 같은 네트워크 안의 컨테이너 통신 |
| NAT | 가능 | 컨테이너의 외부 인터넷 접속 |
| HTTP 요청 흐름 | 가능 | `curl`로 서버 요청 |
| DB 연결 흐름 | 가능 | Spring Boot → PostgreSQL |
| 네트워크 진단 | 가능 | `ping`, `ip`, `ss`, `curl`, `nc` |
| 패킷 분석 | 가능 | `tcpdump` |

특히 현재 공부 중인 Spring Boot, Next.js, PostgreSQL, Docker Compose와 연결하면 실무적인 이해로 이어진다.

---

## 3. 현재 공부와 연결되는 구조

앞으로 목표로 삼을 수 있는 구조는 다음과 같다.

```text
브라우저
  ↓
Next.js
  ↓
Spring Boot API
  ↓
PostgreSQL
```

Docker 네트워크 관점에서는 다음과 같다.

```text
client 컨테이너
  ↓
nginx 컨테이너
  ↓
spring 컨테이너
  ↓
postgres 컨테이너
```

여기서 확인해야 할 질문은 다음과 같다.

```text
Next는 Spring을 어떤 주소로 부르는가?
Spring은 PostgreSQL을 어떤 주소로 부르는가?
localhost는 누구 기준인가?
포트 매핑은 왜 필요한가?
컨테이너 이름으로 접속되는 이유는 무엇인가?
```

---

## 4. 중요한 개념: localhost

Docker를 사용할 때 가장 먼저 헷갈리는 개념은 `localhost`다.

### Windows / 내 PC 기준

```text
localhost = 내 Windows 컴퓨터
```

### Spring 컨테이너 안에서

```text
localhost = Spring 컨테이너 자기 자신
```

그래서 Spring 컨테이너에서 PostgreSQL 컨테이너에 접속할 때 아래 설정은 보통 실패한다.

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/testdb
```

Spring 컨테이너 입장에서 `localhost`는 PostgreSQL이 아니라 Spring 컨테이너 자기 자신이기 때문이다.

Docker Compose나 사용자 정의 Docker Network에서는 보통 이렇게 쓴다.

```properties
spring.datasource.url=jdbc:postgresql://postgres:5432/testdb
```

여기서 `postgres`는 컨테이너 이름 또는 Compose 서비스 이름이다.

---

## 5. 중요한 개념: 포트 매핑

예시:

```yaml
ports:
  - "8080:8080"
```

의미:

```text
내 컴퓨터의 8080번 포트 → 컨테이너의 8080번 포트
```

브라우저에서 다음 주소로 접속하면:

```text
http://localhost:8080
```

실제로는 컨테이너 내부의 8080번 포트로 연결된다.

단, 컨테이너끼리 통신할 때는 보통 `localhost`가 아니라 컨테이너 이름 또는 서비스 이름을 쓴다.

예시:

```text
http://spring:8080
jdbc:postgresql://postgres:5432/testdb
```

---

## 6. 환경 준비

오늘 준비한 기본 환경은 다음과 같다.

```text
Windows
WSL2
Ubuntu
Docker Desktop
Docker WSL Integration
VSCode
```

---

## 7. WSL2 확인

PowerShell에서 확인한다.

```powershell
wsl -l -v
```

정상 예시:

```text
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

중요한 것은 `VERSION`이 `2`인 것이다.

---

## 8. Ubuntu 기본 업데이트

Ubuntu 터미널에서 실행한다.

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 9. 네트워크 실습용 패키지 설치

Ubuntu에 기본 네트워크 도구를 설치한다.

```bash
sudo apt install -y \
  curl \
  wget \
  git \
  vim \
  net-tools \
  iproute2 \
  iputils-ping \
  dnsutils \
  netcat-openbsd \
  tcpdump
```

각 도구의 역할은 다음과 같다.

| 도구 | 역할 |
|---|---|
| `curl` | HTTP 요청 테스트 |
| `wget` | 파일 다운로드 |
| `git` | 코드 저장소 사용 |
| `vim` | 터미널 텍스트 편집 |
| `net-tools` | `ifconfig`, `netstat` 등 구형 네트워크 도구 |
| `iproute2` | `ip addr`, `ip route` 등 현대 Linux 네트워크 도구 |
| `iputils-ping` | `ping` 명령어 |
| `dnsutils` | `dig`, `nslookup` 등 DNS 확인 |
| `netcat-openbsd` | `nc` 명령어로 TCP 포트 확인 |
| `tcpdump` | 패킷 관찰 |

---

## 10. Docker Desktop 설정

Docker Desktop에서 확인할 설정:

```text
Settings
  → General
    → Use the WSL 2 based engine 체크
```

그리고:

```text
Settings
  → Resources
    → WSL Integration
      → Ubuntu 활성화
```

이 설정을 해야 Ubuntu 터미널에서 `docker` 명령어를 사용할 수 있다.

---

## 11. Docker 작동 확인

Ubuntu 터미널에서 실행한다.

```bash
docker version
```

정상이라면 Docker Client와 Docker Server 정보가 모두 나온다.

그다음:

```bash
docker run hello-world
```

정상적으로 실행되면 Docker 환경 준비는 끝이다.

---

## 12. 환경 준비 완료 기준

아래 명령들이 동작하면 준비가 끝난 상태다.

### PowerShell

```powershell
wsl -l -v
```

Ubuntu가 `VERSION 2`로 나와야 한다.

### Ubuntu

```bash
docker version
docker run hello-world
ip addr
curl https://example.com
```

---

# 13. Docker 네트워크 실습 1단계

## 목표

오늘 실습의 핵심 목표는 다음 문장을 이해하는 것이다.

```text
같은 Docker network 안에 있는 컨테이너끼리는
IP가 아니라 이름으로 통신할 수 있다.
```

---

## 14. 기존 실습 환경 정리

이전에 만든 컨테이너가 있으면 삭제한다.

```bash
docker ps -a
```

```bash
docker rm -f linux1 linux2
```

기존 네트워크도 삭제할 수 있다.

```bash
docker network rm net-practice
```

없는 경우 에러가 나도 괜찮다.

---

## 15. Docker 네트워크 생성

```bash
docker network create net-practice
```

확인:

```bash
docker network ls
```

자세히 보기:

```bash
docker network inspect net-practice
```

확인할 항목:

```text
Driver: bridge
Subnet
Gateway
Containers
```

아직 컨테이너를 연결하지 않았다면 `Containers`는 비어 있을 수 있다.

---

## 16. 컨테이너 2개 실행

### linux1 실행

```bash
docker run -itd --name linux1 --network net-practice ubuntu sleep infinity
```

### linux2 실행

```bash
docker run -itd --name linux2 --network net-practice ubuntu sleep infinity
```

확인:

```bash
docker ps
```

정상이라면 `linux1`, `linux2`가 둘 다 실행 중이어야 한다.

---

## 17. 컨테이너 안에 네트워크 도구 설치

### linux1 접속

```bash
docker exec -it linux1 bash
```

컨테이너 안에서 실행:

```bash
apt update
apt install -y iproute2 iputils-ping curl dnsutils netcat-openbsd
```

나오기:

```bash
exit
```

### linux2 접속

```bash
docker exec -it linux2 bash
```

컨테이너 안에서 실행:

```bash
apt update
apt install -y iproute2 iputils-ping curl dnsutils netcat-openbsd
```

나오기:

```bash
exit
```

---

## 18. linux1에서 linux2로 ping

```bash
docker exec -it linux1 bash
```

컨테이너 안에서:

```bash
ping linux2
```

정상 예시:

```text
PING linux2 (172.18.0.3) 56(84) bytes of data.
64 bytes from linux2.net-practice (172.18.0.3): icmp_seq=1 ttl=64 time=0.1 ms
```

멈추기:

```text
Ctrl + C
```

여기서 중요한 점:

```text
linux1이 linux2라는 이름을 IP 주소로 해석했다.
```

즉 Docker 사용자 정의 네트워크 안에서는 컨테이너 이름이 DNS 이름처럼 동작한다.

---

## 19. IP 직접 확인

linux1 안에서:

```bash
ip addr
```

linux2의 IP 확인:

```bash
docker exec -it linux2 ip addr
```

Docker 네트워크 기준으로도 확인할 수 있다.

```bash
docker network inspect net-practice
```

예상 구조:

```json
"Containers": {
  "...": {
    "Name": "linux1",
    "IPv4Address": "172.18.0.2/16"
  },
  "...": {
    "Name": "linux2",
    "IPv4Address": "172.18.0.3/16"
  }
}
```

---

## 20. Docker 내부 DNS 확인

linux1 안에서 확인:

```bash
cat /etc/resolv.conf
```

다음과 비슷하게 나올 수 있다.

```text
nameserver 127.0.0.11
```

의미:

```text
127.0.0.11 = Docker가 컨테이너 내부에 제공하는 DNS 서버
```

그래서 `linux2`라는 이름을 IP 주소로 바꿔줄 수 있다.

이름 해석 확인:

```bash
getent hosts linux2
```

또는:

```bash
nslookup linux2
```

예상:

```text
172.18.0.3 linux2
```

---

# 21. HTTP 서버 실습

## 목표

`linux2`를 서버로 만들고, `linux1`을 클라이언트로 사용한다.

```text
linux1
  ↓ curl
linux2:8000
```

---

## 22. linux2에서 Python HTTP 서버 실행

linux2 접속:

```bash
docker exec -it linux2 bash
```

Python 설치:

```bash
apt update
apt install -y python3
```

테스트 파일 생성:

```bash
echo "hello from linux2" > index.html
```

HTTP 서버 실행:

```bash
python3 -m http.server 8000
```

이 터미널은 서버 실행 상태이므로 그대로 둔다.

---

## 23. linux1에서 linux2로 HTTP 요청

새 Ubuntu/WSL 터미널을 하나 더 열고 실행한다.

```bash
docker exec -it linux1 bash
```

컨테이너 안에서:

```bash
curl http://linux2:8000
```

정상 결과:

```text
hello from linux2
```

이 실습에서 확인한 흐름:

```text
linux1 → linux2:8000 으로 HTTP 요청
Docker 내부 DNS가 linux2를 IP로 변환
linux2의 8000번 포트로 연결
응답이 linux1로 돌아옴
```

---

## 24. 포트 연결 확인: nc

linux1에서 다음 명령어를 사용할 수 있다.

```bash
nc -vz linux2 8000
```

서버가 켜져 있으면:

```text
Connection to linux2 8000 port [tcp/*] succeeded!
```

서버가 꺼져 있으면:

```text
Connection refused
```

구분:

```text
ping 성공 = IP까지 도달 가능
nc 성공 = 해당 TCP 포트까지 연결 가능
curl 성공 = HTTP 응답까지 정상
```

---

# 25. 오늘 이해해야 할 핵심 흐름

```text
ping linux2
```

이 명령이 되는 이유:

```text
linux1 컨테이너
  ↓
Docker 내부 DNS에게 linux2의 IP 요청
  ↓
linux2 = 172.18.0.3 확인
  ↓
같은 bridge network 안에서 패킷 전달
  ↓
linux2 응답
```

---

# 26. 오늘 배운 핵심 개념 요약

## 1. Docker Network

Docker 컨테이너끼리 통신할 수 있게 해주는 가상 네트워크다.

```bash
docker network create net-practice
```

사용자 정의 네트워크를 만들면 컨테이너 이름 기반 통신이 쉬워진다.

---

## 2. Bridge Network

Docker가 Linux의 가상 브리지 구조를 이용해 컨테이너들을 같은 네트워크에 묶는다.

```text
linux1
  ┐
   ├─ Docker bridge network
  ┘
linux2
```

같은 bridge network 안에서는 서로 통신할 수 있다.

---

## 3. Container Name DNS

사용자 정의 Docker 네트워크에서는 컨테이너 이름을 주소처럼 쓸 수 있다.

```bash
ping linux2
curl http://linux2:8000
```

이것이 Spring Boot에서 다음 설정을 쓰는 이유와 연결된다.

```properties
spring.datasource.url=jdbc:postgresql://postgres:5432/testdb
```

---

## 4. localhost의 기준

`localhost`는 항상 “지금 명령을 실행하는 자기 자신”이다.

```text
Windows에서 localhost = Windows 자기 자신
linux1 컨테이너에서 localhost = linux1 자기 자신
spring 컨테이너에서 localhost = spring 자기 자신
postgres 컨테이너에서 localhost = postgres 자기 자신
```

그래서 컨테이너끼리 통신할 때는 보통 `localhost`가 아니라 컨테이너 이름을 쓴다.

---

## 5. ping / nc / curl 차이

| 명령어 | 확인하는 것 |
|---|---|
| `ping linux2` | 네트워크 레벨에서 도달 가능한지 |
| `nc -vz linux2 8000` | TCP 포트가 열려 있는지 |
| `curl http://linux2:8000` | HTTP 응답이 정상인지 |

---

# 27. 다음 실습 방향

다음 순서로 진행하면 좋다.

```text
1. 기본 bridge와 사용자 정의 bridge 차이
2. 포트 매핑: -p 8080:8000
3. PostgreSQL 컨테이너 연결
4. Spring Boot에서 DB_HOST=postgres로 연결
5. Nginx 리버스 프록시
6. Docker Compose로 전체 구조 묶기
```

---

# 28. 다음 실습 예고: 포트 매핑

다음에는 이런 구조를 확인하면 된다.

```text
브라우저 또는 Windows curl
  ↓ localhost:8080
Docker 포트 매핑
  ↓
linux2 컨테이너의 8000번 포트
```

예상 명령어:

```bash
docker run -itd \
  --name web1 \
  --network net-practice \
  -p 8080:8000 \
  ubuntu sleep infinity
```

컨테이너 안에서:

```bash
apt update
apt install -y python3
echo "hello from web1" > index.html
python3 -m http.server 8000
```

Windows 또는 WSL에서:

```bash
curl http://localhost:8080
```

이렇게 하면 “컨테이너 내부 포트”와 “내 PC에서 접근하는 포트”의 차이를 이해할 수 있다.

---

# 29. 오늘의 한 줄 정리

```text
Docker 네트워크 실습의 첫 핵심은
같은 사용자 정의 bridge network 안에서는
컨테이너 이름이 주소처럼 동작한다는 것이다.
```

