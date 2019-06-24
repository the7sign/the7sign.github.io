---
layout: post
title:  "Mysql Procedure 나 function 내 문자열 검색"
date:   2019-06-24 00:00:00
categories: server
tags: mysql procedure function search 
comments: true
---

Mysql 프로시저를 작성하다보면 그 안의 내용을 검색하고 싶을때가 많이 있는데 아래와 같이 information_schema를 이용해서 그 내용을 찾을 수 있다.

``` bash
SELECT ROUTINE_SCHEMA, ROUTINE_NAME FROM information_schema.routines WHERE LOWER(ROUTINE_DEFINITION) LIKE '%_searchword_%';

```

끝