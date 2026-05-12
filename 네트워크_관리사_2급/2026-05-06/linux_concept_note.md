# Linux
리눅스는 정확히 보면 커널이고 우리가 말하는 리눅스 OS는 다음 요소들의 조합이다.
- Linux Kernel
- Shell
- 기본 명령어
- 파일시스템 구조
- 패키지 관리자
- 서비스 관리자
- 설정 파일
- 사용자/권한 시스템

대표적인 배포판은 다음과 같다.
- Ubuntu: 입문자, 서버, WSL에 사용
- Debian: 안정성 중시
- Rocky/Alma Linux: RHEL계열 서버 대체
- Fedora: 최신 기술 반영 빠름
- Arch linux: 직접 구성, 고급 사용자 지향

## 리눅스의 핵심 구조
### 커널
커널은 하드웨어와 프로그램 사이를 중재하는 핵심 부분이다.
예를 들어서 프로그램이 `이 IP로 TCP연결 만들어줘` 라고 한다면
커널이 직접 `네트워크 카드와 TCP/IP 스택을 통해 처리`를 해주게 된다.
### 셸
셸은 사용자의 명령어를 해석해서 실행하는 프로그램이다.
대표적인 셸
- sh: 가장 기본적인 유닉스 셸
- bash: 리눅스에서 가장 흔한 셸
- zsh: 자동완성, 플러그인에 강함
- fish: 사용자 편의성 중심

터미널은 화면이며 셸은 명령어 해석기이다.

## 리눅스 명령어 기본 구조
리눅스 명령어는 보통 아래와 같이 생겼다.
`명령어 [옵션] [대상]` 
예를 들면
`ls -al /home` 
- ls: 목록 보기 명령어
- -al: 옵션
- /home: 대상 경로
와 같다.

### 옵션의 형태
리눅스 명령어 옵션은 보통 두종류가 있다.
`-l`, `-a`
짧은 옵션으로 `-l` `-a` `-la`
긴 옵션으로 `--all` `--help` 등과 같다.
예를 들어 `rm -r folder`에서 `-r`은 recursive, 즉 폴더 안까지 재귀적으로 삭제한다는 뜻이다.

## 자주쓰는 기본 명령어
- pwd: 현재 디렉터리 경로를 출력한다. print working directory를 말한다.
- ls: 파일 목록 보기이다. `-l`을 통해 자세히 보기, `-a`를 통해 숨김 파일까지 보기가 가능하다. `-l`을 할 경우에는 앞부분에는 파일 형태 (`d`: 디렉터리, `-`: 일반 파일, `l`: 심볼릭 링크), 그 뒤에는 권한이 가온다.
- cd: 디렉터리 이동이다. (패스)
- cat: 파일 내용 보기. 
- less: 페이지 단위로 보기
- head: 앞부분 보기 `-n` 을 통해 앞의 특정 개수의 줄을 볼 수 있다.
- tail: 뒷부분 보기 `-n`을 통해 끝부분부터 지정한 라인 수만큼 출력 가능
- `tail -f 파일명`: 로그 실시간 보기
- touch: 파일 만들기
- mkdir: 디렉터리 만들기. `-p`를 통해 중첩 디렉터리를 만들 수 있다.
- cp a b: a를 b로 파일 복사
- cp -r a b: 폴더 a를 b에 복사
- mv a b: a의 이름을 b로 변경 또는 b로 이동
- rm: 파일 삭제
- rm -r: 폴더 삭제
- rm -rf: 강제삭제

## 경로 개념
절대경로는 `/` 루트부터 시작하는 경로이다.
상대경로는 현재 위치 기준이다. `./` 

## 리눅스 파일시스템 구조
리눅스는 모든것이 `/` 아래에 붙는다.
- bin
- boot
- dev
- etc
- home 
- lib
- media
- mnt
- opt
- proc
- root
- run 
- sbin
- tmp
- usr
- var
### 종류
- `/`: 루트 디렉터리로 리눅스 파일시스템의 최상위다.
- `/home`: 일반 사용자들의 홈 디렉터리가 있다. `/home/user`, `/home/hurwan0629` 와같이. 개인 피일, 프로젝트, 설정 파일이 보통 여기에 있다.
- `/root`: root 사용자의 홈 디렉터리이다. 일반 사용자의 홈인 `/home/user`와 다르다.
- `/etc`: 시스템 설정 파일들이 들어있다. `/hosts`, `/passwd`, `/group`, `/ssh/sshd_config`, `/nginx/nginx.conf`, `/systemd/system` 등이 있다. (웹서버, SSH, 사용자 정보, 네트워크 설정 등)
- `/var`: 변하는 데이터가 들어간다. (대표적으로 로그)
- `/usr`: 사용자용 프로그램, 라이브러리, 공유 파일들이 있다. `/bin`, `/sbin`, `/lib`, `/share`. 특히 `/bin`에 많은 명령어가 존재한다. (python3, curl, git ...)
- `/bin`, `/sbin`: 기본 명령어들이 들어있다. `/bin`에는 일반 사용자도 쓰는 기본 명령어, `/sbin`에는 시스템 관리용 명령어가 들어간다. 현대 리눅스에는 `/bin`이 `/usr/bin`에 들어가는 경우도 많다.
- `/dev`: 장치 파일이 들어있다. 리눅스에서는 하드웨어 장치도 파일처럼 다룬다. 특히 `/dev/null`은 버리는 구멍같은 특수 파일이다.
- `/proc`: 커널과 프로세스 정보를 보여주는 가상 파일시스템이다. 실제 디스크에 저장된 파일 보다는 실시간으로 보여주는 정보이다. `cat /proc/cpuinfo`를 통해 cpu정보, `cat /proc/meminfo`를 통해 메모리 정보를 확인 가능하다.
- `/tmp`: 임시 파일 저장소이다. 프로그램들이 임시 데이터를 저장하며 재부팅 시 삭제될 수 있다.
- `/opt`: 추가 설치 프로그램들이 들어가는 경우가 많다. 예를 들어 `/opt/google`, `/opt/idea` 등이 있다.
- `/mnt`, `/media`: 외부 저장 장치나 다른 파일 시스템을 마운트할 때 쓴다. WSL에서는 윈도우 C드라이브가 보통 `/mnt/c`에 존재한다.

## 권한 시스템
리눅스는 다중 사용자 OS이기 때문에 파일마다 소유자와 권한이 있다.
- `ls -l`을 통해 [파일타입] [소유자] [그룹] [외부자] 접근 권한을 볼 수 있다.
- `chmod [u][g][o] filename`: 권한 변경 명령어이다.
- `sudo chown 소유자:그룹 파일`: 해당 파일의 소유자와 그룹 파일을 설정한다.
- `sudo`: 관리자 권한으로 명령을 실행한다. `/etc`, `/var`, 서비스 설정 등은 일반 사용자 권한으로 수정할 수 없는 경우가 많다.
- `whoami`: 현재 사용자 확인
- `id`: 사용자 id 정보 -> (`uid=1000(user) gid=1000(user) groups=1000(user),27(sudo)`)
- `/etc/passwd`: 사용자 목록이 존재한다. `user:x:1000:1000:User:/home/user:/bin/bash`는 
    - `user`: 사용자 명
    - `x`: 비밀번호는 `/etc/shadow`에 존재
    - `1000`: uid
    - `1000`: gid
    - `User`: 설명
    - `/home/user`: 홈 디렉터리
    - `bin/bash`: 기본 셸
- `/etc/group`: 그룹 권한의 묶음으로 예를 들어 `sudo` 그룹에 속하면 관리자 명령을 사용할 수 있다.
## 패키지 관리
- `sudo apt update`: 패키지 목록을 최신화한다.
- `sudo apt upgrade`: 실제 설치된 패키지를 업데이트한다
- `sudo install package`: 패키지 설치
- `sudo apt remove package`: 패키지 삭제
- `sudo apt purge package`: 설정 파일까지 제거
- `sudo apt autoremove`: 불필요한 의존성 제거
- `apt search package`: 패키지 검색
- `apt show package`: 패키지 정보 보기
## 프로세스 관리
- `px aux`: 프로세스 목록 보기. `a`: all, `u`: user, `x`: 터미널에 없는 서비스
- `px aux | grep process`: 특정 프로세스 찾기
- `top`: 실시간 프로세스 보기
- `htop`: 패키지를 통한 편의 `top`
- `kill PID`: 프로세스 종료
- `kill -9 PID`: 프로세스 강제 종료
## 서비스 관리 (systemd)
`systemctl`을 통한 서비스 관리
- `systemctl status service`: 서비스 상태 확인
- `sudo systemmctl [start/stop/restart] service`: 서비스 시작/중지/재시작
- `sudo systemctl enable service`: 부팅시 자동 실행
- `sudo systemctl disable service`: 부팅시 자동 실행 해제
- `journalctl -u service -f`: 서비스 로그 실시간 보기 `-u`는 unit의 약자, `-f`(follow) `-n`을 통한 최근 로그 확인 가능
## 네트워크 명령어
- `ip addr`(`ip a`): IP 확인
- `ip route`: 라우팅 확인
- `ping ip`: 연결 확인
- `nslookup alias`(`dig alias`): DNS 조회. `sudo apt install dnsutils`를 통해 설치
## 포트 확인
- `ss -tulnp`: 열린(Listening) 포트 확인 `-tu`: TCP와 UDP, `-l`: Listening, `-n`: numeric로 서비스 이름 대신 숫자로 표시(결과가 빨리나옴), `-p`: 프로세스까지 확인, `| grep :포트`로 특정 포트 조회 가능
- `curl -i [url]`: HTTP 요청 테스트. `-i`를 통해 헤더까지 확인 가능.
- `curl -X [HTTP메서드] [url] -H "[헤더]" -d '{데이터}'`: HTTP 구체적 요청
## 리다이렉션과 파이프
프로그램에는 기본 입출력이 있다.
- stdin - 0: 표준 입력
- stdout - 1: 표준 출력
- stderr - 2: 표준 에러
- `echo hello > a.txt`: 덮어씌우기
- `echo hello >> a.txt`: 파일에 추가하기
- `command > out.txt 2>err.txt`: 에러까지 저장. (stdout는 out.txt로, stder는 err.txt)에 저장한다.
### 파이프
파이프는 `|`를 통해 명령어의 출력을 뒤 명령어의 입력으로 넘긴다.
- `ps aux | grep java`: 프로세스 목록 출력 -> 그중 java가 들어간 줄만 필터링
## 검색 명령어
- `grep [검색어] [file]`: file에서 검색어 검색. `-i`를 통해 대소문자 무시, `-n`을 통해 줄 번호 표시, `-r`을 통해 하위 폴더까지 검색
- `find . -name "*.log"`: 파일 이름으로 검색. `-type d`를 통해 디렉터리만 검색 가능 (f하면 파일만), `-size`를 통해 +100M 같은 크기 기준도 가능
- `which`: 명령어 위치 확인 (`whereis`)
## 압축과 아카이브
- `.tar`: 여러 파일을 하나로 묶음
- `.gz`: gzip 압축
- `.tar.gz`: 묶고 gzip 압축
- `.zip`: zip 압축

- `tar -czvf [압축명].tar.gz [플더]/`: `*.tar.gz` 만들기. c: create, z: gzip, v: verbose(과정 보여주기), f: file이름 지정
- `tar -xzvf [파일명].tar.gz`: `*.tar.gz` 풀기. `x`는 extract입니다. `t`는 내용물만 확인하는 것입니다.
## 사용자
리눅스느느 기본적으로 다중 사용자 OS이다.
한 컴퓨터 안에서 여러 사용자가 존재할 수 있고, 각 사용자마다 권한이 다르다.
- root사용자: 리눅스의 최고 관리자 계정. 관리자 계정보다 더 강한 권한을 가짐. 모든 파일과 설정을 수정할 수 있으며, 일반적으로 `sudo`를 통해서 관리자 권한을 빌린다.
- 일반 사용자: 일반 사용자는 사람이 로그인해서 사용하는 계정으로 Ubuntu 설치할 때 만든 사용자 계정이 이에 해당한다. 자기 홈 디렉터리 안에서 자유롭게 파일을 만들 수 있다.
- 시스템 사용자: 사람이 직접 로그인 하지 않는 사용자들. daemon, bin, sys, www-data, systemd-network, sshd, messsagebus등이 있다. 이런 계정들은 특정 프로그램이나 서비스 실행용 계정이다. 이는 웹 서버가 해킹당했을 때 피해 범위를 줄이기 위해서이다.
- 그룹: 리눅스 권한은 사용자만으로 관리하지 않으며 그룹도 같이 사용한다. `groups`를 통해 사용자가 속한 그룹을 확인 가능하다.
## 사용자 정보 파일
사용자와 그룹 정보는 `/etc` 속 설정 파일에 저장된다.
- `/passwd`: 사용자 목록.
- `/shadow`: 비밀번호 해시
- `/group`: 그룹 목록
- `/sudoers`: sudo 권한 설정