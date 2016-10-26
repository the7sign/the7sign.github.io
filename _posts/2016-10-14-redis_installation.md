---
layout: post
title:  "Redis Installation"
date:   2016-10-14 00:00:00
categories: server
tags: redis install
comments: true
---

Redis 설치방법에 대해서 정리해보겠습니다.

## 다운로드

먼저 <http://redis.io> 에 접속하여 최신 stable 버전을 다운받는다.

``` bash
wget http://download.redis.io/releases/redis-3.2.0.tar.gz
tar -xvzf redis-3.2.0.tar.gz
cd redis-3.2.0

make
make install PREFIX=/data/apps/redis3.2.0 #make install 시에 prefix를 통해서 설치 디렉토리를 정해준다.
```

소스 폴더의 utils 디렉토리로 이동해서 install_server.sh 를 실행한다.

``` bash
Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379]
Selecting default: 6379

Please select the redis config file name [/etc/redis/6379.conf] /data/apps/redis3.2.0/redis.conf
Please select the redis log file name [/var/log/redis_6379.log] /data/apps/redis3.2.0/log/redis.log
Please select the data directory for this instance [/var/lib/redis/6379] /data/apps/redis3.2.0/data
Please select the redis executable path [] /data/apps/redis3.2.0/bin/redis-server

Selected config:
Port           : 6379
Config file    : /data/apps/redis3.2.0/redis.conf
Log file       : /data/apps/redis3.2.0/log/redis.log
Data dir       : /data/apps/redis3.2.0/data
Executable     : /data/apps/redis3.2.0/bin/redis-server
Cli Executable : /data/apps/redis3.2.0/bin/redis-cli

Is this ok? Then press ENTER to go on or Ctrl-C to abort.

Copied /tmp/6379.conf => /etc/init.d/redis_6379

Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!
```
위의 설정값은 예시이니 편한대로 설정해서 사용하면 된다.

이 설정이 끝나면 아래와 같이 서비스의 시작/종료를 할수 있다.

``` bash
service redis_10001 stop;
service redis_10001 start;
```

## 추가설정

/etc/logrotate.conf 에 설정 추가

``` bash
/data/apps/redis3.2.0/log/*.log {
       weekly
       rotate 10
       copytruncate
       delaycompress
       compress
       notifempty
       missingok
}
```

/etc/sysctl.conf 에 아래 내용 추가

``` bash
# Redis M-S sync
vm.overcommit_memory=1

# Increase max open file limit
fs.file-max = 1048576
```

## redis.conf 

MASTER 설정

``` bash
bind 61.38.75.240

# slaveof 10.0.10.3 6379 # Master 는 slaveof 설정이 없어야 한다
masterauth 1234%^&*
requirepass 1234%^&*
```

SLAVE 설정

``` bash
bind 61.38.75.241

slaveof 61.38.75.240 10001
masterauth 1234%^&*
requirepass 1234%^&*
```

패스워드를 입력했으면 실행 파일에 대해서 아래와 같이 수정한다. (/etc/init.d/redis_10001)

``` bash
PASSWD="1234%^&*"

....

//$CLIEXEC -p $REDISPORT shutdown
$CLIEXEC -p $REDISPORT -a $PASSWD shutdown
```

접속 및 테스트

``` bash
redis-cli -h 127.0.0.1 -p 10001

127.0.0.1:10001> AUTH 1234%^&* => 미리 설정된 비밀 번호로 인증
OK
127.0.0.1:10001> set "testkey" "testvalue"
OK
127.0.0.1:10001> get "testkey"
"testvalue"
```

> **서버추가**  
> - REDIS의 경우 싱글 쓰레드형태이기 때문에 일반적으로 코어1개당 1개의 서비스를 구동하여 사용한다.  
> - 이때에는 위에 설정된 값중 conf 파일과 실행파일 부분만 한개씩 추가하여 만들고 구성하면 된다.  
