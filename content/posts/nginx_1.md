---
Title: "엔진엑스(Nginx) 주요 설정 파일과 간단한 명령어"
date: 2023-06-16T23:57:18+09:00
draft: true
---

## 1. 주요 설정 파일, 디렉터리 및 명령어 이해하기

### 1.1 엔진엑스 주요 설정 파일과 디렉터리 구조

- **/etc/nginx**: 엔진엑스 서버가 사용하는 기본 설정이 저장된 루트 디렉터리
- **/etc/nginx/nginx.conf**: 엔진엑스의 기본 설정 파일로, 글로벌 설정 항목을 포함
- **/etc/nginx/conf.d**: 기본 HTTP 서버 설정 파일이 포함된 디렉터리
- **/var/log/nginx**: 엔진엑스의 로그가 저장되는 디렉터리

### 1.2 주요 엔진엑스 명령어

- **nginx -h**: 엔진엑스 도움말 확인
- **nginx -v**: 엔진엑스 버전 정보 확인
- **nginx -V**: 엔진엑스 빌드 정보 및 모듈 확인
- **nginx -T**: 엔진엑스 설정 시험 결과 확인
- **nginx -s signal**: 지정된 신호를 엔진엑스 마스터 프로세스에 전송 (stop, quit, reload, reopen)

## 2. 정적 콘텐츠 서비스 설정하기

다음은 엔진엑스로 정적 콘텐츠를 제공하는 기본 설정 예시

```
server {
    listen 80 default_server;
    server_name www.example.com;
    location / {
        root /usr/share/nginx/html;
        # alias /usr/share/nginx/html;
        index index.html index.htm;
    }
}
```

위 설정은 HTTP 프로토콜과 80 포트를 이용해 정적 콘텐츠를 제공합니다. `root` 지시자를 사용하여 제공할 콘텐츠의 위치를 지정할 수 있으며, `index` 지시자로 기본 파일 지정이 가능합니다.

## 3. 무중단 설정 리로드

엔진엑스 설정을 변경할 때 서버를 중지하지 않고 설정을 리로드하는 방법은 다음과 같습니다.

```
nginx -s reload
```

이 명령은 동작 중인 엔진엑스의 마스터 프로세스에 리로드 신호를 보내 설정을 다시 읽도록 지시합니다. 결과적으로 패킷 손실 없이 엔진엑스 설정을 변경하며, 이 기능을 활용하여 무중단 설정 리로드가 가능합니다. 

이상으로 엔진엑스의 주요 설정 파일, 디렉터리, 명령어 이해 및 정적 콘텐츠 제공 설정 방법과 무중단 설정 리로드에 대해 살펴보았습니다. 이를 통해 엔진엑스를 사용할 준비가 되었습니다. 변경된 설정을 시험하고 문제가 없다면, `nginx -s reload` 명령으로 무중단 설정 리로드를 수행하세요.