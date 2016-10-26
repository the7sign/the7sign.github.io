---
layout: post
title:  "Git Credential"
date:   2016-10-26 00:00:00
categories: git
tags: git credential
comments: true
---

Git을 사용하다 보면 ID/PWD를 물어보는 경우가 있는데 이를 한번 입력으로 자동처리 하는 방법에 대해서 알아보겠습니다.

## Git Credential 실행

    
    $ git credential fill   #이 명령으로 인증정보를 가지고 오는 과정을 시작한다
    protocol=https
    host=unknownhost
    
    Username for 'https://unknownhost': bob
    Password for 'https://bob@unknownhost':
    protocol=https
    host=unknownhost
    username=bob
    password=s3cre7

protocol과 host를 입력한 후 엔터를 치면 인증 정보를 확인하고 그 정보를 바탕으로 앞으로 사용된다.
