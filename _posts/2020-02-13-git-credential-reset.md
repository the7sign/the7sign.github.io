---
layout: post
title:  "Git Credential 초기화 방법"
date:   2020-02-13 00:00:00
categories: dev
tags: git credential 
comments: true
---

Git를 사용중에 인증하는 계정이 삭제 되게나 변경되게 되면 인증 오류메시지를 내면서 pull이나 push에 문제가 발생하는 경우가 있습니다.

이럴때는 고민하지 마시고 아래의 명령어로 기존에 저장되어있던 Credential 정보를 모두 삭제하고 다시 시작하는게 가장 빠릅니다.

## Credential Remove

``` bash
git credential-manager uninstall

```
실행하면 아래와 같이 메시지가 나오면서 초기화 된다.

``` bash
Looking for Git installation(s)...
  C:\Program Files\Git

Updated your /etc/gitconfig [git config --system]
Updated your ~/.gitconfig [git config --global]

Removing from 'C:\Program Files\Git'.
  removal failed. U_U

Press any key to continue...
```

이후에 git pull이나 push를 실행하면 다시 계정을 묻게 되고 저장 할 수 있다.

끝