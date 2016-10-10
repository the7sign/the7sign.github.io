---
layout: post
title:  "MariaDB Installation"
date:   2016-10-10 00:00:00
categories: server
tags: mariadb install
comments: true
---

PHP 설치 방법을 정리하겠습니다.

## 기본 디렉토리 생성

``` bash
mkdir -p /data /data/apps /data/webtmps /data/www /data/log /data/src
```

## 의존성 패키지 설치

``` bash
yum -y install cmake make gcc gcc-c++ ncurses-devel libevent openssl openssl-devel libxml2 libxml2-devel bison wget
```

## 유저및그룹 생성

``` bash
useradd -M -r -s /sbin/nologin maria
```

## MARIADB 다운로드 및 압축해제

버전은 최신 stable 버전을 설치해주시면 됩니다.

``` bash
cd /data/src
wget https://downloads.mariadb.org/interstitial/mariadb-10.1.16/source/mariadb-10.1.16.tar.gz/from/http%3A//ftp.kaist.ac.kr/mariadb/
tar xvzf mariadb-10.1.16.tar.gz
```

## MARIADB configure

``` bash
cd /data/src/mariadb-10.1.16

cmake .. \
-DCMAKE_INSTALL_PREFIX=/data/apps/mariadb \
-DINSTALL_SYSCONFDIR=/data/apps/mariadb/etc \
-DINSTALL_LIBDIR=/data/apps/mariadb/lib64 \
-DINSTALL_SUPPORTFILESDIR=/data/apps/mariadb/support-files \
-DTMPDIR=/data/apps/mariadb/tmp \
-DPID_FILE_DIR=/data/apps/mariadb \
-DDAEMON_NAME=mariadb \
-DMYSQL_TCP_PORT=3306 \
-DMYSQL_DATADIR=/data/apps/mariadb/data \
-DMYSQL_UNIX_ADDR=/data/apps/mariadb/mysql.sock \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_READLINE=1 \
-DWITH_SSL=bundled \
-DWITH_ZLIB=system \
-DWITH_JEMALLOC=no \
-DWITH_EXTRA_CHARSETS=all \
-DWITH_ARIA_STORAGE_ENGINE=1 \
-DWITH_XTRADB_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_PERFSCHEMA_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_FEDERATEDX_STORAGE_ENGINE=1 \
-DWITH_QUERY_CACHE_INFO=1 \
-DWITH_QUERY_RESPONSE_TIME=1 \
-DWITH_SAFEMALLOC=AUTO \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci

make

make install
```

## tmp 및 logs 디렉토리 생성 & 디렉토리 초기화

``` bash
mkdir -p /data/apps/mariadb/tmp /data/apps/mariadb/logs /data/apps/mariadb/binlog /data/apps/mariadb/iblog
rm -Rf /data/apps/mariadb/data/*
chown -Rf maria.maria /data/apps/mariadb
```

## my.cnf 설정

``` bash
cat /dev/null > /data/apps/mariadb/etc/my.cnf

#vi 편집기로 /data/apps/mariadb/etc/my.cnf 파일 연 후 아래 내용 붙여넣기

[client]
port                            = 3306
socket                          = /data/apps/mariadb/mysql.sock
[mysql]
no_auto_rehash
[myisamchk]
aria_pagecache_buffer_size      = 64M
sort_buffer_size                = 64M
read_buffer                     = 2M
write_buffer                    = 2M
[mysqlhotcopy]
interactive-timeout
[mysqldump]
quick
max_allowed_packet              = 1024M
default-character-set           = utf8
[mysqld_safe]
open_files_limit                = 8192
user                            = maria
[mysqld]
user                            = maria
port                            = 3306
default-time-zone               = SYSTEM
socket                          = /data/apps/mariadb/mysql.sock
# GENERAL #
default_storage_engine          = InnoDB
pid-file                        = /data/apps/mariadb/mysqld.pid
transaction_isolation           = REPEATABLE-READ
event_scheduler                 = ON
log_warnings                    = 2
# CHARACTER SET #
init_connect                    = SET collation_connection = utf8_general_ci
character-set-server            = utf8
collation-server                = utf8_general_ci
# DATA STORAGE #
#basedir                        =
datadir                        = /data/apps/mariadb/data
tmpdir                          = /data/apps/mariadb/tmp
# SAFETY #
skip_name_resolve               = 1
# CACHES AND LIMITS #
connect_timeout                 = 20
max_connections                 = 1000
max_user_connections            = 0
sort_buffer_size                = 4M
read_rnd_buffer_size            = 2M   # 8 -> 2
read_buffer_size                = 2M   # 8 -> 2
join_buffer_size                = 2M   # 8 -> 2
binlog_cache_size               = 1M
max_heap_table_size             = 64M
tmp_table_size                  = 64M
thread_cache_size               = 64
thread_handling                 = pool-of-threads
thread_pool_max_threads         = 1024
thread_pool_idle_timeout        = 256
thread_stack                    = 256K
table_definition_cache          = 4096
table_open_cache                = 4096
query_cache_type                = ON
query_cache_size                = 32M
query_cache_limit               = 2M
max_allowed_packet              = 1024M
wait_timeout                    = 10
performance_schema              = 0
# MyISAM #
myisam_sort_buffer_size         = 64M
# INNODB #
innodb_data_home_dir            = /data/apps/mariadb/data
innodb_log_group_home_dir       = /data/apps/mariadb/logs/iblog
innodb_data_file_path           = ibdata1:2048M;ibdata2:2048M:autoextend
innodb_autoextend_increment     = 100
innodb_flush_method             = O_DIRECT
innodb_file_per_table           = 1
innodb_buffer_pool_size         = 12G
innodb_log_file_size            = 256M
innodb_log_buffer_size          = 4M
innodb_thread_concurrency        = 0  #쓰레드 폴 사용으로 0 지정
#innodb_log_files_in_group       = 3  #default 2
innodb_flush_log_at_trx_commit  = 1

# LOGGING #
log-error                       = /data/apps/mariadb/logs/err_log.log
slow_query_log_file             = /data/apps/mariadb/logs/slow_log.log
slow_query_log                  = 1
long_query_time                 = 1
# BINARY LOGGING #
binlog_format                   = mixed
expire_logs_days                = 7
log-bin                         = /data/apps/mariadb/logs/binlog/mysql-bin
log-bin-index                   = /data/apps/mariadb/logs/binlog/bin-log.index
max_binlog_size                 = 1G
log_bin_trust_function_creators = 1

# REPLICATION #
server-id                      = 1   #Master 서버 ID 지정
#relay-log                      =
#relay-log-index                =
#relay-log-info-file            =
#master-info-file               =
```

## data 디렉토리 초기화

``` bash
cd /data/apps/mariadb/scripts
./mysql_install_db --basedir=/data/apps/mariadb --datadir=/data/apps/mariadb/data --user=maria
```
아래와 같은 형식으로 OK가 떨어져야 정상 / 실패하면 failed 라고 나옴
``` bash
Installing MySQL system tables...
090720 14:01:07 [Warning] Forcing shutdown of 2 plugins
OK
Filling help tables...
090720 14:01:07 [Warning] Forcing shutdown of 2 plugins
OK
To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system
PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:
./bin/mysqladmin -u root password 'new-password'
./bin/mysqladmin -u root -h localhost.localdomain password 'new-password'
Alternatively you can run:
./bin/mysql_secure_installation
which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.
See the manual for more instructions.
You can start the MySQL daemon with:
cd . ; ./bin/mysqld_safe &
You can test the MySQL daemon with mysql-test-run.pl
cd ./mysql-test ; perl mysql-test-run.pl
Please report any problems with the ./bin/mysqlbug script!
```

## MARIADB 패스워드 설정을 위한 실행

``` bash
/data/apps/mariadb/bin/mysqld_safe &

/data/apps/mariadb/bin/mysql_secure_installation

* 간략 설명
Enter current password for root (enter for none): ## 그냥 엔터
Set root password? [Y/n] ## root 패스워드를 설정 할 것인지 물어본다.
Remove anonymous users? [Y/n] ##  anonymous 유저를 삭제 할것인지 물어본다.
Disallow root login remotely? [Y/n] ## root의 원격 접속을 허용할 것인지 물어본다.
Remove test database and access to it? [Y/n] ## test 데이터베이스를 삭제 할것인지 물어본다.
Reload privilege tables now? [Y/n] ## privileges 테이블을 재 시작 할 것인지 물어본다.

```

## 자동 시작 스크립트 생성 및 등록

``` bash
cp /data/apps/mariadb/etc/init.d/mysql /etc/init.d/mysqld
chmod 755 /etc/init.d/mysqld
chkconfig --add mysqld
```

## 실행

``` bash
service mysqld start
service mysqld stop
/etc/init.d/mysqld start
/etc/init.d/mysqld stop
```