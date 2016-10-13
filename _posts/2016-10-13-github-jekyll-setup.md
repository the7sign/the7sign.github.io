---
layout: post
title:  "GitHub & Jekyll Setup"
date:   2016-10-12 00:00:00
categories: web
tags: github jekyll setup
comments: true

---

깃허브를 이용해 마크다운 블로그를 만드는 방법을 찾다가 jekyll 이라는 걸 확인하고 아래와 같이 설정해서 사용중입니다.

(참고로 윈도우10 기준입니다)

## Ruby & Ruby Devkit 설치

http://rubyinstaller.org/downloads/ 에 접속 해서 최신 버전의 파일을 다운받는다.

작성일 기준으로 rubyinstaller-2.3.1-x64.exe / DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe 


![루비설치화면](/images/20150911_jelky_install_02.jpg "이미지제목")

위와 같이 설치중 가운데 부분의 PATH 설정을 체크 한 후 사용한다. (안해서 설치는 되지만 후에 개발중 편의를 위해서)

DevKit의 경우 적절한 위치에 압축을 푼다. (ex C:\RubyDevKit)

## Jekyll 설치

콘솔창을 RubyDevKit 위치에서 열고 아래와 같이 입력한다.

``` bash 
gem install jekyll
```

> **Note**

> - SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://api.rubygems.org/specs.4.8.gz)
> - 설치중에 위의 메시지처럼 SSL인증 오류가 발생하는 케이스가 있는데 이럴때는 아래와 같은 형태로 install 작업을 진행한다.
> - gem install jekyll --source http://rubygems.org

