---
layout: post
title:  "Haproxy Installation"
date:   2016-10-17 00:00:00
categories: server
tags: haproxy install
comments: true
---

Redis 설치방법에 대해서 정리해보겠습니다.

## 다운로드 & 컴파일

먼저 <http://www.haproxy.org/> 에 접속하여 최신 stable 버전을 다운받는다.

``` bash
wget http://www.haproxy.org/download/1.6/src/haproxy-1.6.9.tar.gz
tar -xvzf haproxy-1.6.9.tar.gz

cd haproxy-1.6.9

make TARGET=linux26   # LINUX 종류에 따라서 값이 다름

make DESTDIR="/data/apps/haproxy" PREFIX="" install
```
## haproxy.cfg 파일 생성

아래 설정 파일은 REDIS HAPROXY 설정값으로 내용은 대상 서버의 종류에 따라서 변경이 된다.

또한 TCP 서버나 HTTP서버도 설정이 가능하다.

``` bash
cd /data/apps/haproxy
vi haproxy.cfg
```
아래의 값을 입력한다.
``` bash
global
    log 127.0.0.1       local2
        stats socket /tmp/haproxy mode 600 level admin

defaults        REDIS
 log    global
 mode   tcp
 balance        roundrobin
 maxconn        2000
 option tcplog
 timeout connect  4s
 timeout server  15s
 timeout client  15s
# timeout tunnel 365d

frontend ft_redis_master
 bind *:5000 name redis
 default_backend bk_redis_master

backend bk_redis_master
 option tcp-check
 tcp-check send AUTH\ 1234%^&*\r\n
 tcp-check expect string +OK
 tcp-check send PING\r\n
 tcp-check expect string +PONG
 tcp-check send info\ replication\r\n
 tcp-check expect string role:master
 tcp-check send QUIT\r\n
 tcp-check expect string +OK
 server REDIS_10001 127.0.0.1:10001 check inter 3s
 server REDIS_10002 127.0.0.1:10002 check inter 3s

frontend ft_redis_slave
 bind *:5001 name redis
 default_backend bk_redis_slave

backend bk_redis_slave
 option tcp-check
 tcp-check send AUTH\ 1234%^&*\r\n
 tcp-check expect string +OK
 tcp-check send PING\r\n
 tcp-check expect string +PONG
 tcp-check send info\ replication\r\n
 tcp-check expect string role:slave
 tcp-check send QUIT\r\n
 tcp-check expect string +OK
 server REDIS_10001 127.0.0.1:10001 check inter 3s
 server REDIS_10002 127.0.0.1:10002 check inter 3s
```

## 시작 스크립트 작성

``` bash
cp examples/haproxy.init /etc/init.d/haproxy
chmod 755 /etc/init.d/haproxy

# vi 편집기로 /etc/init.d/haproxy 파일 연 후 실행 파일등에 대한 경로설정을 수정한다.

BIN=/data/apps/haproxy/sbin/$BASENAME # 이런 경로를 수정한다.
CFG=/data/apps/$BASENAME/$BASENAME.cfg # 이런 경로를 수정한다.
```

## 로그 설정

아래와 같이 파일을 추가한다.

``` bash
touch /etc/rsyslog.d/haproxy.conf
```

생성된 파일에 아래 값을 입력한다.

``` bash
$ModLoad imudp
$UDPServerAddress   127.0.0.1
$UDPServerRun   514
$template Haproxy,"%msg%\n"
local2.=info -/var/log/haproxy.log;Haproxy
local2.notice -/var/log/haproxy-status.log;Haproxy
### keep logs in localhost ##
local2.* ~
```
> **로그관련 주의사항**  
> - log파일 경로를 /var/log 이외의 경로로 지정은 가능하나 머신별로 /var/log 이외에 로그가 생성되지 않는 경우가 있었습니다.     
> - /var/log에 생성후 링크 파일로 가지고 오는 것을 권장 합니다.  

# rsyslog 재시작
``` bash
/etc/init.d/rsyslog restart
```

## 실행
``` bash
/etc/init.d/haproxy start
/etc/init.d/haproxy stop
```