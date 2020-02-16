---
layout: post
title: "[오픈소스테스팅 with Angora] libxml Automata"
tags: [Fuzzing, Angora, 캡스톤]
comments: true
---

> 앙고라로 뻐징하기  

## build  
먼저 target program libxml의 configuration status를 Angora와 맞춰주어야 한다.  
~~~
CC=/home/ysh/Angora/bin/angora-clang CXX=/home/ysh/Angora/bin/angora-clang++ LD=/home/ysh/Angora/bin/angora-clang ./configure --disable-shared
~~~

libxml이 zlib을 사용하는데 Angora에서 taint tracking 과정에서 이 부분을 제외해야 돌아간다. 이에 대한 설명은 [여기](https://github.com/AngoraFuzzer/Angora/blob/master/docs/example.md)에 자세히 나와있다.  

fast, taint version에 대해서 다음과 같이 make를 한다.  
~~~
USE_TRACK=1 make -j
~~~
~~~
make clean
USE_FAST=1 make -j
~~~

## run Angora  
준비가 되었으므로 libxml 중 automata 프로그램을 타겟으로 fuzzing 해보자!  
~~~
./angora_fuzzer -i ../libxml2-2.9.2_fast/test/automata/ -o libxml -t ../libxml2-2.9.2_taint/testAutomata -- ../libxml2-2.9.2_fast/testAutomata @@
~~~

![Center example image](https://user-images.githubusercontent.com/35067611/72702255-60e4e180-3b95-11ea-845d-11b865c5f079.png "Center"){: .center-image}  

귀여운 토끼 앙고라가 나온다. 약 23시간 테스팅한 결과로 110개의 crash를 발견했음을 알 수 있다.  

### debugging  
이제 이 프로그램에 crash를 발생시키는 input들이 위에 지정한 output 폴더에 저장되어있을 것이다. 이것들로 automata 프로그램을 실행시켜서 어떤 종류의 crash인지 알아보고 gdb로 어디서 발생하는지도 알아보자.  

![Center example image](https://user-images.githubusercontent.com/35067611/72702356-bb7e3d80-3b95-11ea-8029-7e44554eee6a.png "Center"){: .center-image}  

segmentation fault를 발생시키는 id:000040 파일.  

![Center example image](https://user-images.githubusercontent.com/35067611/72702654-d3a28c80-3b96-11ea-9c15-780d7cfb14d1.png "Center"){: .center-image}  

xmlregexp.c:3953 에서 seg fault가 발생한 것을 찾아냈다!  