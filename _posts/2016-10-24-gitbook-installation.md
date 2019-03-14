---
layout: post
title:  "GitBook 설치 방법"
date:   2016-10-24 00:00:00
categories: server
tags: git gitbook install
comments: true
---

gitbook 설치 방법을 정리하겠습니다.

## 의존성 설치

``` bash
yum install gcc openssl-devel gcc-c++ compat-gcc-34 compat-gcc-34-c++
```

## nodeJS 설치

``` bash
wget https://nodejs.org/dist/v4.6.1/node-v4.6.1.tar.gz
tar -xf node-v4.6.1.tar.gz
cd node-v4.6.1
./configure --prefix=/usr/local/node
make
make install

ln -s /usr/local/node/bin/* /usr/sbin/  #링크를 설정
```

## npm설치

``` bash
curl -L https://npmjs.org/install.sh | sh
```

## git 설치

``` bash
yum install git

git config --global user.name Jack
git config --global user.email jack@xx.com
git config --global core.editor vim
git config --global merge.tool vimdiff
git config --list
```

## gitbook 설치

``` bash
npm install -g gitbook
npm install -g gitbook-cli

gitbook -V
```

> TIP
> - 로컬 작업을 위해서 윈도우에서 설치시에도 각 해당하는 프로그램(nodeJS, Git)을 윈도우버전을 받아 설치한다.
> - 그리고 다른 명령어는 동일하게 사용이 가능하다.

