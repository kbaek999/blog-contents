---
title: "고성능 부하분산을 위한 엔진엑스 활용 방법-2 (TCP, UDP)"
date: 2023-06-20T22:27:18+09:00
tags: [nginx]
categories: [Tech]
---

## 엔진엑스 TCP 부하분산

엔진엑스는 HTTP뿐만 아니라 TCP, UDP 트래픽을 위한 부하분산도 지원합니다. TCP 부하분산은 업스트림 블록과 stream 모듈을 사용하여 구현됩니다.

```
upstream mysql-example.read {
  server read1.mysql-example.com:3306 weight=5;
  server read2.mysql-example.com:3306;
  server 172.0.0.22:3306 mysql-backup;
}

server {
  listen 3306;
  proxy_pass mysql-example.read;
}
```

위 설정은 3306 포트로 TCP 요청을 받아 읽기 전용 MySQL 복제본 두 대로 부하를 분산합니다. 프라이머리 MySQL 서버 두 대가 모두 다운되면 backup 매개변수로 지정한 서버로 요청을 전달합니다.

엔진엑스 설치 전후 특별히 설정을 바꾸지 않았다면 엔진엑스의 기본 설정 파일 경로인 conf.d 폴더는 http 블록에 포함됩니다. 따라서 stream 모듈을 이용한 이 설정은 stream.conf.d 라는 별도의 폴더를 생성해 저장하는 편이 좋습니다. 경로를 nginx.conf 파일의 stream 블록에 추가해 엔진엑스가 참조하도록 합니다.

```
/etc/nginx/nginx.conf 설정 파일:

user nginx;
worker_processes auto;
pid /run/nginx.pid;

stream {
  include /etc/nginx/stream.conf.d/*.conf;
}

/etc/nginx/stream.conf.d/mysql_read-conf 설정 파일:

upstream mysql-example.read {
  server read1.mysql-example.com:3306 weight=5;
  server read2.mysql-example.com:3306;
  server 172.0.0.22:3306 mysql-backup;
}

server {
  listen 3306;
  proxy_pass mysql-example.read;
}
```

## 엔진엑스 UDP 부하분산

UDP 부하분산은 TCP와 마찬가지로 stream 모듈을 통해 설정하며 방법도 매우 비슷합니다. 가장 큰 차이점은 listen 지시자를 통해 UDP 데이터그램을 처리할 소켓을 지정한다는 점입니다. 데이터그램을 다룰 때는 TCP 부하분산에서 사용하지 않는 지시자를 몇 가지 사용합니다. 대표적으로 엔진엑스가 업스트림 서버로부터 수신할 것으로 예상되는 응답의 크기를 지정하는 데 proxy_response 지시자를 사용합니다. 이 지시자를 설정하지 않으면 proxy_timeout 지시자의 제한값이 되기 전까지 무제한으로 응답을 처리합니다. proxy_timeout은 연결을 닫기 전에 목적지 서버로의 읽기, 쓰기 작업 완료를 기다리는 시간을 지정하는 데 사용합니다.

```
stream {
  server {
    listen 1195 udp reuseport;
    proxy_pass 127.0.0.1:1194;
  }
}
```

reuseport 매개변수는 엔진엑스가 워커 프로세스별로 개별 수신 소켓을 만들어 사용하도록 합니다. 커널은 엔진엑스로 보내야 하는 연결들을 워커 프로세스 단위로 분산하고, 따라서 클라이언트와 서버가 주고받는 여러 패킷을 동시에 처리할 수 있습니다.