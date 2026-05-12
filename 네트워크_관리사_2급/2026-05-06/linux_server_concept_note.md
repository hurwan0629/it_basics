# 서버 운영 기본기
## Apache
Apache HTTP Server은 대표적인 웹 서버 프로그램이다.
역할은 간단히 말하면
`클라이언트 요청 -> Apache가 요청 수신 -> 정적 파일 반환 또는 백엔드 서버로 프록시 -> 응답 전송`과 같다.
Apache는 보통 80번 포트에서 요청을 받는다.
HTTPS라면 443번 포트를 이용한다.
주요 역할은 다음과 같다.
- 정적 파일 제공
- 가상 호스트
- 리버스 프록시
- SSL/TLS처리 
- 로그 기록

`sudo apt update`, `sudo apt install apache2`를 통해 설치 가능하다.

`/etc/httpd/conf/httpd.conf`에서 메인 설정을 할 수 있다.

## Well-known port
Well-known port는 1~1023 포트에서 잘 알려진 포트이지만 속하지 않은 포트도 종종 포함된다

Linux에서 1024 미만 포트는 privileged port로 일반 사용자가 바로 열 수 없으며 root 권한으로 직접 식행하거나 Apache/Nginx가 받고 내부 앱으로 프록시 하는 방법이 있다.

ufw 때문에 외부에서 접속이 되지 않을수도 있다.

## 네트워크 