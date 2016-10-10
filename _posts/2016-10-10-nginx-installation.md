---
layout: post
title:  "Nginx Installation"
date:   2016-10-10 00:00:00
categories: server
tags: nginx install
comments: true
---

Nginx 설치 방법을 정리하겠습니다.

## 기본 디렉토리 생성

``` bash
mkdir -p /data /data/apps /data/webtmps /data/www /data/log /data/src
```

## 의존성 패키지 설치

``` bash
yum -y install pcre-devel openssl openssl-devel make gcc gcc-c++ wget
```

## 유저및그룹 생성

``` bash
useradd -M -r -s /sbin/nologin webuser
```

## nginx 다운로드 및 압축해제

버전은 최신 버전을 설치해주시면 됩니다.

``` bash
cd /data/src
wget http://nginx.org/download/nginx-1.11.3.tar.gz
tar xvzf nginx-1.11.3.tar.gz
```

## nginx configure

``` bash
cd /data/src/nginx-1.11.3

./configure \
--prefix=/data/apps/nginx \
--sbin-path=/data/apps/nginx/sbin/nginx \
--pid-path=/data/apps/nginx \
--conf-path=/data/apps/nginx/conf/nginx.conf \
--user=webuser \
--group=webuser \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_v2_module \
--http-client-body-temp-path=/data/apps/nginx/_client_body_temp \
--http-proxy-temp-path=/data/apps/nginx/_proxy_temp \
--http-fastcgi-temp-path=/data/apps/nginx/_fastcgi_temp \
--http-uwsgi-temp-path=/data/apps/nginx/_uwsgi_temp \
--http-scgi-temp-path=/data/apps/nginx/_scgi_temp \
--with-ipv6

#위 옵션에서 --with-http_v2_module 은 --with-http_spdy_module 로 교체 가능함.
make

make install
```

## 디렉토리 정리 및 config 초기화

``` bash
chown webuser.webuser -Rf /data/webtmps
rm -Rf /data/apps/nginx/conf/*.default
mkdir /data/apps/nginx/conf/sites-available /data/apps/nginx/conf/sites-enabled
cat /dev/null > /data/apps/nginx/conf/nginx.conf

## sites-available 디렉토리에는 site config 파일을 작성해 넣고,
sites-enabled 디렉토리에는 sites-available 디렉토리에 작성한 파일 중 실제로 서비스할 파일만 ln 명령어로 링크만 걸어두는 식으로 관리하는게 편함
ex) ln -s /data/apps/nginx/conf/sites-available/www.example.com.conf /data/apps/nginx/conf/sites-enabled/www.example.com.conf
```

## 기본 설정 파일

``` bash
user                        webuser;
worker_processes            2; ## worker_processes 부분에 CPU 코어 갯수를 적는다.
worker_rlimit_nofile        15000;

error_log               logs/error.log  crit;
pid                     /data/apps/nginx/nginx.pid;

events {
      worker_connections    15000;
      use epoll;
      multi_accept          on;
}

http {
      include           mime.types;
      default_type      application/octet-stream;

      log_format        main    '$remote_addr - $remote_user [$time_local] "$request" '
                                '$status $body_bytes_sent "$http_referer" '
                                '"$http_user_agent" "$http_x_forwarded_for"';

      sendfile        on;
      tcp_nopush      on;
      tcp_nodelay     on;

      keepalive_timeout                 65;
      reset_timedout_connection         on;

      types_hash_max_size               2048;
      server_names_hash_bucket_size     256;

      client_max_body_size              32k;
      client_body_buffer_size           32k;
      client_body_in_single_buffer      on;
      client_body_timeout               180s;
      client_header_timeout             180s;
      client_header_buffer_size         32k;
      large_client_header_buffers       4 32k;

      gzip                              on;
      gzip_http_version                 1.1;
      gzip_vary                         on;
      gzip_comp_level                   9;
      gzip_proxied                      any;
      gzip_types                        text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;
      gzip_buffers                      16 8k;
      gzip_disable                      "MSIE [1-6]\.(?!.*SV1)";

      server_tokens   off;

      include sites-enabled/*;
}
```

## 자동 시작 스크립트 작성

``` bash
touch /etc/init.d/nginx
chmod 755 /etc/init.d/nginx

vi 편집기로 /etc/init.d/nginx 파일 연 후 아래 내용 붙여넣기

#!/bin/sh
#
# nginx - this script starts and stops the nginx daemin
#
# chkconfig: - 85 15
# description: Nginx is an HTTP(S) server, HTTP(S) reverse \
# proxy and IMAP/POP3 proxy server
# processname: nginx
# config: /data/apps/nginx/conf/nginx.conf
# pidfile: /data/apps/nginx/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/data/apps/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/data/apps/nginx/conf/nginx.conf"

lockfile=/var/lock/subsys/nginx

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    start
}


reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
    $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}


case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
    rh_status_q || exit 0
    ;;
    *)
    echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
exit 2
esac
```

## 자동시작 스크립트 등록

``` bash
chkconfig nginx on
```

## 로그 설정

``` bash
ln -s /data/apps/nginx/logs /data/log/nginx

vi 편집기로 /etc/logrotate.d/nginx 파일 연 후 아래 내용 붙여넣기

/data/apps/nginx/logs/*.log {
    daily
    missingok
    rotate 31
    compress
    notifempty
    sharedscripts
    postrotate
        [ ! -f /data/apps/nginx/nginx.pid ] || kill -USR1 `cat /data/apps/nginx/nginx.pid`
    endscript
    copytruncate
}
```

## 실행

``` bash
service nginx start
service nginx stop
/etc/init.d/nginx start
/etc/init.d/nginx stop
```

## WebSite Conf

### Codeigniter

``` bash
server {
    listen      80;
    server_name www.abc.com;

    charset     utf-8;

    access_log  logs/www.abc.com_access.log  main;
    error_log   logs/www.abc.com_error.log   crit;

    root        /data/www/www.abc.com;
    index       index.php;

    client_max_body_size    1024M;

    # enforce NO www
    if ($host ~* ^www\.(.*))
    {
        set $host_without_www $1;
        rewrite ^/(.*)$ $scheme://$host_without_www/$1 permanent;
    }

    # canonicalize codeigniter url end points
    # if your default controller is something other than "welcome" you should change the following

    # removes trailing slashes (prevents SEO duplicate content issues)
    if (!-d $request_filename)
    {
        rewrite ^/(.+)/$ /$1 permanent;
    }

    # removes access to "system" folder, also allows a "System.php" controller
    if ($request_uri ~* ^/system)
    {
        rewrite ^/(.*)$ /index.php?/$1 last;
        break;
    }

    # unless the request is for a valid file (image, js, css, etc.), send to bootstrap
    if (!-e $request_filename)
    {
        rewrite ^/(.*)$ /index.php?/$1 last;
        break;
    }

    location ~ \.php$ {
        fastcgi_pass                    unix:/data/apps/php/php-fpm.sock;
        fastcgi_index                   index.php;
        fastcgi_buffers                 256 16k;
        fastcgi_buffer_size             128k;
        fastcgi_connect_timeout         180s;
        fastcgi_send_timeout            600;
        fastcgi_read_timeout            600;
        fastcgi_busy_buffers_size       256k;
        fastcgi_temp_file_write_size    256k;
        fastcgi_max_temp_file_size      0;
        fastcgi_intercept_errors        on;

        fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param   PHP_VALUE       mysql.default_socket=/data/apps/mariadb/mysql.sock;
        fastcgi_param   PHP_VALUE       mysqli.default_socket=/data/apps/mariadb/mysql.sock;

        include fastcgi_params;
    }

    location ~* \.(?:ico|css|js|gif|jpe?g|png)$ {
        expires max;
        access_log      off;
        log_not_found   off;

        add_header Pragma public;
        add_header Cache-Control "public, must-revalidate, proxy-revalidate";
    }

        # deny access to apache .htaccess files
    location ~ /\.ht
    {
        deny all;
    }

}
```

### Nova Framework

``` bash
server {
        listen      80;
        server_name www.abc.com;

        charset     utf-8;

        access_log  logs/www.abc.com_access.log main;
        error_log   logs/www.abc.com_error.log  crit;

        root        /data/www/www.abc.com;
        index       index.php index.html;

        client_max_body_size    10M;

        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';

        location / {
                try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
                try_files $uri = 404;
                fastcgi_pass                    unix:/data/apps/php/php-fpm.sock;
                fastcgi_split_path_info ^(.+.php)(/.+)$;
                #fastcgi_index                   index.php;
                fastcgi_buffers                 1024 64k;
                fastcgi_buffer_size             256k;
                fastcgi_connect_timeout         300s;
                fastcgi_send_timeout            900;
                fastcgi_read_timeout            900;
                fastcgi_busy_buffers_size       256k;
                fastcgi_temp_file_write_size    256k;
                fastcgi_max_temp_file_size      0;
                fastcgi_intercept_errors        on;

                fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;

                include fastcgi_params;
        }

        location = /favicon.ico {
                access_log      off;
                log_not_found   off;
        }

        location ~* \.(?:icc|css|js|gif|jpe?g|png)$ {
                expires max;
                access_log      off;
                log_not_found   off;

                add_header Pragma public;
                add_header Cache-Control "public, must-revalidate, proxy-revalidate";
        }

        location ~ /\.ht {
                deny all;
         }
}
```

## SSL 설정

listen 과 charset 사이에 아래의 구문을 추가한다. 물론 인증서는 별도로 존재해야 한다.

``` bash
ssl on;
ssl_certificate     /etc/pki/tls/certs/domainname.crt;
ssl_certificate_key /etc/pki/tls/certs/domainname.key;
ssl_session_timeout 5m;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';

ssl_prefer_server_ciphers on;
ssl_dhparam     /etc/pki/tls/certs/dhparam.pem;
ssl_session_cache shared:SSL:50m;
```

## IPV6

아래와 같이 각 사이트 conf에 설정한다.

``` bash
listen 80; ## listen for ipv4; this line is default and implied
listen [::]:80 default_server ipv6only=on; ## listen for ipv6
server_name     iosqapbm.goodgames.net;
```

아래와 같이 netstat 를 통해서 설정된 포트를 확인 가능하다.

``` bash
netstat -nlp | grep nginx
tcp        0      0 0.0.0.0:8080                0.0.0.0:*                   LISTEN      24635/nginx
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      24635/nginx
tcp        0      0 0.0.0.0:8081                0.0.0.0:*                   LISTEN      24635/nginx
tcp        0      0 0.0.0.0:443                 0.0.0.0:*                   LISTEN      24635/nginx
tcp        0      0 :::8080                     :::*                        LISTEN      24635/nginx
tcp        0      0 :::80                       :::*                        LISTEN      24635/nginx
```