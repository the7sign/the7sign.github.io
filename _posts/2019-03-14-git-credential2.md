---
layout: post
title:  "Git 사용자 인증 문제 (비밀번호 변경등)"
date:   2019-03-14 18:16:00
categories: git
tags: git credential
comments: true
---

Git을 사용하는 중 계정의 비밀번호가 변경이 되면 아래와 같은 메시지와 함께 Push나 Pull이 되지 않습니다.

    fatal: remote error: Invalid username or password.

여러가지 방법을 다 써봐도 안되서 Bash에서 아래와 같이 remote정보를 삭제한 후 다시 추가 하니 정상적으로 사용이 가능했습니다.

    git remote remove origin
    git remote add origin https://username:userpasswd@repository_url
    
각 저장소 별로 다 해줘야 하는게 문제긴 하지만 빠르게 오류를 수정할 수 있었습니다.\
다른 방법이 또 있으면 수정해서 포스팅 할께요
