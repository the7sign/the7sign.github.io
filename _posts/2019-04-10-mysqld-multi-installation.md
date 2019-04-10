---
layout: post
title:  "MariaDB 여러 인스턴스 한서버에 띠우기"
date:   2019-04-10 19:15:00
categories: server
tags: mysql mariadb
comments: true
---

서버 개발을 하다보면 자원의 한계나 편의를 위해서 db서버 1대에 여러개의 인스턴스를 띄우는 경우가 있는데 이번에 그 내용을 정리해 보겠습니다.\
먼저 MariaDB 최신버전을 설치 합니다.

보통 yum으로 설치하면 최신버전을 찾지 못하는 경우가 있는데 이럴때는 직접 설정 파일을 수정합니다.

    /etc/yum.repos.d/maraidb.repo
    
해당 파일이 없으면 새로 만든후 아래와 같이 저장한다.

    [mariadb]
    name = MariaDB
    baseurl = http://yum.mariadb.org/10.3/rhel7-amd64
    gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
    gpgcheck=1

이후에 설치를 진행합니다.

    yum install mariadb mariadb-server    
    
그럼 설치가 끝나게 됩니다.

이제 여러개 인스턴스를 실행하는 스크립트를 작성합시다.

    vi /etc/my.cnf.d/mysql_client.conf
    
해당 파일을 열어서 아래와 같은 항목을 추가합시다.

    [mysqld1]
    user		= mysql
    pid-file	= /var/run/mysqld/mysqld3307.pid
    socket		= /var/run/mysqld/mysqld3307.sock # socket 기반 접속 안할경우 불필요
    port		= 3307
    basedir		= /usr
    datadir		= /var/lib/mysql3307
    tmpdir		= /tmp
    log-error	= /var/log/mysql/error3307.log
    
    [mysqld2]
    user		= mysql
    pid-file	= /var/run/mysqld/mysqld3308.pid
    socket		= /var/run/mysqld/mysqld3308.sock # socket 기반 접속 안할경우 불필요
    port		= 3308
    basedir		= /usr
    datadir		= /var/lib/mysql3308
    tmpdir		= /tmp
    log-error	= /var/log/mysql/error3308.log
    
[mysqld1] 이부분은 나중에 실행 할때 사용하게 됩니다. 우린 2개를 설정 할거니까 2개를 넣었습니다.

그리고 폴더를 만들고 권한을 넣어 줍니다.

    mkdir /var/lib/mysql3307
    mkdir /var/lib/mysql3308
    mkdir /var/log/mysql
    
    chown -R mysql.mysql /var/lib/mysql3307
    chown -R mysql.mysql /var/lib/mysql3308
    chown -R mysql.mysql /var/log/mysql
    
이제 마지막으로 데이터베이스 초기화를 해줍니다.

    mysql_install_db --user=mysql --datadir=/var/lib/mysql3307 --basedir=/usr
    mysql_install_db --user=mysql --datadir=/var/lib/mysql3308 --basedir=/usr

이제 설정이 다 끝났으니 실행을 하면 됩니다.

    mysqld_multi start 1,2      -- 실행
    mysqld_multi stop 1,2       -- 중지
    
    
    