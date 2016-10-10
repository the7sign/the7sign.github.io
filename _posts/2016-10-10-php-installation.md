---
layout: post
title:  "PHP-FPM Installation"
date:   2016-10-10 00:00:00
categories: server
tags: php install
comments: true
---

PHP 설치 방법을 정리하겠습니다.

## 기본 디렉토리 생성

``` bash
mkdir -p /data /data/apps /data/webtmps /data/www /data/log /data/src
```

## 의존성 패키지 설치

``` bash
yum -y install openssl openssl-devel libxml2 libxml2-devel zlib-devel make gcc gcc-c++ wget
```

## 유저및그룹 생성

``` bash
useradd -M -r -s /sbin/nologin webuser
```

## php7 다운로드 및 압축해제

버전은 최신 stable 버전을 설치해주시면 됩니다.

``` bash
cd /data/src
wget http://kr1.php.net/get/php-7.0.10.tar.gz/from/this/mirror -O php-7.0.10.tar.gz
tar xvzf php-7.0.10.tar.gz
```

## php configure

``` bash
cd /data/src/php-7.0.10

./configure \
--prefix=/data/apps/php \
--enable-fpm \
--with-openssl \
--with-zlib \
--with-config-file-path=/data/apps/php/etc/ \
--with-mysqli=mysqlnd \
--with-pdo_mysql=mysqlnd

make

make install
```

## php.ini 파일 작성

``` bash
cp /data/src/php-7.0.10/php.ini-production /data/apps/php/etc/php.ini

vi 편집기로 /data/apps/php/etc/php.ini 연 후 아래 내용 검색 후 수정
expose_php = Off
cgi.fix_pathinfo=0
allow_url_fopen = Off
date.timezone = Asia/Seoul # 국가에 맞게
upload_tmp_dir = /data/webtmps
session.save_path = "/data/webtmps"
soap.wsdl_cache_dir="/data/webtmps"
```

## php-fpm.conf 수정

vi /data/apps/php/etc/php-fpm.conf 아래 내용 추가 (일부 옵션값은 적절히 수정)

``` bash
[global]
pid = run/php-fpm.pid
error_log = log/php-fpm.log
emergency_restart_interval = 0s
emergency_restart_threshold = 0
process_control_timeout = 0s
rlimit_files = 102400
rlimit_core = unlimited

[www]
user = webuser
group = webuser

listen.mode = 0660
listen = /data/apps/php/php-fpm.sock
listen.allowed_clients = 127.0.0.1
listen.mode = 0660
listen.owner = webuser
listen.group = webuser

pm = ondemand
pm.max_children = 1000
pm.process_idle_timeout = 10s
pm.status_path = /php-fpm_status

php_value[short_open_tag] = On
php_value[memory_limit] = 512M
php_value[max_execution_time] = 0
php_value[max_input_time] = -1
php_value[post_max_size] = 1024M
php_value[upload_max_filesize] = 1024M

request_terminate_timeout = 120s
catch_workers_output = yes

request_slowlog_timeout = 5s
slowlog = var/log/php-slow.log
```

## 자동 시작 스크립트 작성

``` bash
touch /etc/init.d/php-fpm
chmod 755 /etc/init.d/php-fpm

vi 편집기로 /etc/init.d/php-fpm 파일 연 후 아래 내용 붙여넣기

#! /bin/sh

### BEGIN INIT INFO
# Provides:          php-fpm
# Required-Start:    $remote_fs $network
# Required-Stop:     $remote_fs $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts php-fpm
# Description:       starts the PHP FastCGI Process Manager daemon
### END INIT INFO

prefix=/data/apps/php
exec_prefix=${prefix}

php_fpm_BIN=${exec_prefix}/sbin/php-fpm
php_fpm_CONF=${prefix}/etc/php-fpm.conf
php_fpm_PID=${prefix}/var/run/php-fpm.pid


php_opts="--fpm-config $php_fpm_CONF --pid $php_fpm_PID"


wait_for_pid () {
    try=0

    while test $try -lt 35 ; do

            case "$1" in
                    'created')
                    if [ -f "$2" ] ; then
                            try=''
                            break
                    fi
                    ;;

                    'removed')
                    if [ ! -f "$2" ] ; then
                            try=''
                            break
                    fi
                    ;;
            esac

            echo -n .
            try=`expr $try + 1`
            sleep 1

    done

}

case "$1" in
    start)
            echo -n "Starting php-fpm "

            $php_fpm_BIN --daemonize $php_opts

            if [ "$?" != 0 ] ; then
                    echo " failed"
                    exit 1
            fi

            wait_for_pid created $php_fpm_PID

            if [ -n "$try" ] ; then
                    echo " failed"
                    exit 1
            else
                    echo " done"
            fi
    ;;

    stop)
            echo -n "Gracefully shutting down php-fpm "

            if [ ! -r $php_fpm_PID ] ; then
                    echo "warning, no pid file found - php-fpm is not running ?"
                    exit 1
            fi

            kill -QUIT `cat $php_fpm_PID`

            wait_for_pid removed $php_fpm_PID

            if [ -n "$try" ] ; then
                    echo " failed. Use force-quit"
                    exit 1
            else
                    echo " done"
            fi
    ;;

    status)
            if [ ! -r $php_fpm_PID ] ; then
                    echo "php-fpm is stopped"
                    exit 0
            fi

            PID=`cat $php_fpm_PID`
            if ps -p $PID | grep -q $PID; then
                    echo "php-fpm (pid $PID) is running..."
            else
                    echo "php-fpm dead but pid file exists"
            fi
    ;;

    force-quit)
            echo -n "Terminating php-fpm "

            if [ ! -r $php_fpm_PID ] ; then
                    echo "warning, no pid file found - php-fpm is not running ?"
                    exit 1
            fi

            kill -TERM `cat $php_fpm_PID`

            wait_for_pid removed $php_fpm_PID

            if [ -n "$try" ] ; then
                    echo " failed"
                    exit 1
            else
                    echo " done"
            fi
    ;;

    restart)
            $0 stop
            $0 start
    ;;

    reload)

            echo -n "Reload service php-fpm "

            if [ ! -r $php_fpm_PID ] ; then
                    echo "warning, no pid file found - php-fpm is not running ?"
                    exit 1
            fi

            kill -USR2 `cat $php_fpm_PID`

            echo " done"
    ;;

    *)
            echo "Usage: $0 {start|stop|force-quit|restart|reload|status}"
            exit 1
    ;;

esac
```

## 자동시작 스크립트 등록

``` bash
chkconfig php-fpm on
```

## 로그 설정

``` bash
ln -s /data/apps/php/var/log /data/log/php

#vi 편집기로 /etc/logrotate.d/php-fpm 파일 연 후 아래 내용 붙여넣기

/data/apps/php/var/log/*.log {
    daily
    missingok
    rotate 31
    compress
    notifempty
    sharedscripts
    postrotate
        [ ! -f /data/apps/php/var/run/php-fpm.pid ] || kill -USR1 `cat /data/apps/php/var/run/php-fpm.pid`
    endscript
    copytruncate
}
```

## 실행

``` bash
service php-fpm start
service php-fpm stop
/etc/init.d/php-fpm start
/etc/init.d/php-fpm stop
```

## Extenstions

### curl

``` bash
cd /data/src/php-7.0.10/ext/curl #해당 폴더에 기본적인 extension파일이 존재함.
/data/apps/php/bin/phpize
./configure --with-php-config=/data/apps/php/bin/php-config
make
make install

#vi 편집기로 /data/apps/php/etc/php.ini 연 후 맨 아래에 아래내용 추가
extension=curl.so

service php-fpm reload (php를 재구동 해줘야 적용됨)
```
### igbinary

``` bash
wget https://github.com/igbinary/igbinary7/archive/master.zip
unzip master.zip
cd igbinary7-master

/data/apps/php/bin/phpize
./configure --with-php-config=/data/apps/php/bin/php-config

make
make install

#vi 편집기로 /data/apps/php/etc/php.ini 연 후 맨 아래에 아래내용 추가
extension=igbinary.so

service php-fpm reload (php를 재구동 해줘야 적용됨)
```

### mbstring

``` bash
cd /data/src/php-7.0.10/ext/mbstring #해당 폴더에 기본적인 extension파일이 존재함.

/data/apps/php/bin/phpize
./configure --with-php-config=/data/apps/php/bin/php-config

make
make install

#vi 편집기로 /data/apps/php/etc/php.ini 연 후 맨 아래에 아래내용 추가
extension=mbstring.so

service php-fpm reload (php를 재구동 해줘야 적용됨)
```

### mcrypt

``` bash
cd /data/src/php-7.0.10/ext/mcrypt #해당 폴더에 기본적인 extension파일이 존재함.

yum -y install epel-release
yum -y install php-mcrypt
yum -y install libmcrypt-devel

/data/apps/php/bin/phpize
./configure --with-php-config=/data/apps/php/bin/php-config

make
make install

#vi 편집기로 /data/apps/php/etc/php.ini 연 후 맨 아래에 아래내용 추가
extension=mcrypt.so

service php-fpm reload (php를 재구동 해줘야 적용됨)
```

### redis

``` bash
wget https://github.com/phpredis/phpredis/archive/php7.zip
unzip php7.zip
cd phpredis-php7

/data/apps/php/bin/phpize
./configure --with-php-config=/data/apps/php/bin/php-config

make
make install

#vi 편집기로 /data/apps/php/etc/php.ini 연 후 맨 아래에 아래내용 추가
extension=mcrypt.so

service php-fpm reload (php를 재구동 해줘야 적용됨)
```