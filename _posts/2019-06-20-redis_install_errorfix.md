---
layout: post
title:  "Redis 설치시 오류 수정"
date:   2019-06-20 00:00:00
categories: server
tags: redis install
comments: true
---

Redis 설치시에 서버의 환경에 따라서 오류가 나는데 이때 수정하는 방법을 정리한다.

먼저 <http://redis.io> 에 접속하여 최신 stable 버전을 다운받고 압축을 푼다.

``` bash
wget http://download.redis.io/releases/redis-5.0.5.tar.gz
tar -xvzf redis-5.0.5.tar.gz
cd redis-5.0.5

make
```

gcc가 설치가 안되어있으면 아래와 같이 오류가 난다.

``` bash
cd src && make all
make[1]: Entering directory `/data/src/redis-5.0.5/src'
    CC Makefile.dep
make[1]: Leaving directory `/data/src/redis-5.0.5/src'
make[1]: Entering directory `/data/src/redis-5.0.5/src'
rm -rf redis-server redis-sentinel redis-cli redis-benchmark redis-check-rdb redis-check-aof *.o *.gcda *.gcno *.gcov redis.info lcov-html Makefile.dep dict-benchmark
... 중략 ....
MAKE hiredis
cd hiredis && make static
make[3]: Entering directory `/data/src/redis-5.0.5/deps/hiredis'
gcc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  net.c
make[3]: gcc: Command not found
make[3]: *** [net.o] Error 127
make[3]: Leaving directory `/data/src/redis-5.0.5/deps/hiredis'
make[2]: *** [hiredis] Error 2
make[2]: Leaving directory `/data/src/redis-5.0.5/deps'
make[1]: [persist-settings] Error 2 (ignored)
    CC adlist.o
/bin/sh: cc: command not found
make[1]: *** [adlist.o] Error 127
make[1]: Leaving directory `/data/src/redis-5.0.5/src'
make: *** [all] Error 2

```
그럼 yum install gcc 로 설치해준다. 

그럼 make clean해 준 후에 재설치를 하면 잘 되야 하는데 아래와 같이 오류가 난다.

``` bash
[root@outlawqa02 redis-5.0.5]# make clean
cd src && make clean
make[1]: Entering directory `/data/src/redis-5.0.5/src'
rm -rf redis-server redis-sentinel redis-cli redis-benchmark redis-check-rdb redis-check-aof *.o *.gcda *.gcno *.gcov redis.info lcov-html Makefile.dep dict-benchmark
make[1]: Leaving directory `/data/src/redis-5.0.5/src'
[root@outlawqa02 redis-5.0.5]# make
cd src && make all
make[1]: Entering directory `/data/src/redis-5.0.5/src'
    CC Makefile.dep
make[1]: Leaving directory `/data/src/redis-5.0.5/src'
make[1]: Entering directory `/data/src/redis-5.0.5/src'
    CC adlist.o
In file included from adlist.c:34:0:
zmalloc.h:50:31: fatal error: jemalloc/jemalloc.h: No such file or directory
 #include <jemalloc/jemalloc.h>
                               ^
compilation terminated.
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory `/data/src/redis-5.0.5/src'
make: *** [all] Error 2

```

jamalloc 에서 문제가 생긴건데 아무리 다시해도 안된다...ㄷㄷㄷ

의외로 해결책은 간단한데 make clean이 아니라 make distclean을 해주면 된다.

``` bash
[root@outlawqa02 redis-5.0.5]# make distclean
cd src && make distclean
make[1]: Entering directory `/data/src/redis-5.0.5/src'
rm -rf redis-server redis-sentinel redis-cli redis-benchmark redis-check-rdb redis-check-aof *.o *.gcda *.gcno *.gcov redis.info lcov-html Makefile.dep dict-benchmark
(cd ../deps && make distclean)
make[2]: Entering directory `/data/src/redis-5.0.5/deps'
(cd hiredis && make clean) > /dev/null || true
(cd linenoise && make clean) > /dev/null || true
(cd lua && make clean) > /dev/null || true
(cd jemalloc && [ -f Makefile ] && make distclean) > /dev/null || true
(rm -f .make-*)
make[2]: Leaving directory `/data/src/redis-5.0.5/deps'
(rm -f .make-*)
make[1]: Leaving directory `/data/src/redis-5.0.5/src'
```

끝