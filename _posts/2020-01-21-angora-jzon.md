---
layout: post
title: "[오픈소스테스팅 with Angora] Jzon"
tags: [Fuzzing, Angora, 캡스톤]
comments: true
---

> 앙고라로 뻐징하기  

[Jzon](https://github.com/Zguy/Jzon.git)은 json parser로 뻐징해보기 좋은 타겟프로그램이다.  

## build  
이 오픈소스는 configure 파일이 없으므로 직접 makefile에서 다음과 같이 경로를 잡아줘야 했다.  
~~~
CXX=/home/ysh/Angora/bin/angora-clang++
~~~

그리고 taint, fast version으로 각각 make를 하고 Angora로 돌린다.  
~~~
USE_TRACK=1 make -j
~~~
~~~
make clean
USE_FAST=1 make -j
~~~

## run Angora  
Angora로 돌린 command는 다음과 같다. 참고로 마지막 @@는 input의 형태가 file의 경우 붙여주는 옵션인데 AFL과 동일하다.  
~~~
./angora_fuzzer -i ../Jzon_fast/test/spec/ -o jzon -t ../Jzon_taint/test/bin/test -- ../Jzon_fast/test/bin/test @@
~~~

![Center example image](https://user-images.githubusercontent.com/35067611/72702984-bcb06a00-3b97-11ea-9f87-29bde93a0eb5.png "Center"){: .center-image}  

Jzon은 AFL로 하루 이상 돌렸을 때도 crash가 없었는데 Angora도 마찬가지이다. 참 잘 만들었나보다..(?)