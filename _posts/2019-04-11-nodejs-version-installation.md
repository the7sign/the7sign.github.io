---
layout: post
title:  "Node.js 특정/최신 버전 설치하기"
date:   2019-04-11 17:10:00
categories: server
tags: nodejs npm
comments: true
---

NodeJs를 CentOs에 설치하다보면 최신버전이 아닌 이상한 0.10 과 같은 버전이 기본적으로 선택되서 설치됩니다. 

이때 최신버전을 설치하는 방법입니다.

    curl --silent --location https://rpm.nodesource.com/setup_11.x | bash

각각의 버전은 setup_11.x 부분을 수정하여 적용하면 됩니다.

    yum install nodejs
    
그럼 정상적으로 우리가 선택한 버전이 나오며 설치하면 됩니다.

npm의 경우 아래와 같이 업그레이드 시킵니다.

    npm install -g npm
    
    
    