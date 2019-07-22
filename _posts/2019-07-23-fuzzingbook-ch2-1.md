---
layout: post
title: "Software Testing Study CH 2-1"
tags: [Software Testing, Fuzzing, 공부]
comments: true
---

> Fuzzing: Breaking Things with Random Inputs  

둘째 장의 첫 토픽은 `Fuzzing`에 대한 간략한 소개와 배경설명이다. `Fuzzing` 이란 무엇인며 어떻게 내부적으로 구현되어있는지 정리한다. 또한 이를 이용해서 `Software Testing`에 어떻게 적용하는지 공부해보았다.  

### What is Fuzzing?  
invalid, unexpected, random data 를 입력으로 받아 버그 테스팅을 하는 기술을 말한다. 비가 많이 오고 천둥이 치던 밤 우연히 발견되었는데 뇌우가 컴퓨터를 연결해놓은 전화선에 노이즈를 발생시켰고 이것이 UNIX command를 crash 하거나 bad input을 집어넣었다. 그 결과 놀랍게도 프로그램이 더욱 견고해져서 이를 과학적으로 조사하기 위해서 fuzzing 이라는 기술이 발전하기 시작했다.  

가장 간단한 fuzz generator로는 random characters가 있다. implementation은 다음과 같다.  
~~~python
def fuzzer(max_length=100, char_start=32, char_range=32):
    string_length = random.randrange(0, max_length + 1)
    out = ""
    for i in range(0, string_length):
        out += chr(random.randrange(char_start, char_start + char_range))
    return out
~~~
크게 어려운 알고리즘은 없다. 이제 이 fuzzer를 이용해서 여러 경우에 대해 테스팅을 해 볼 것이다. 다만 모든 과정과 코드를 설명하기에는 포스팅이 너무 길어지므로 핵심적인 내용만 짚고 넘어가려 한다.  

1. Fuzzing External Programs  
실제 프로그램에 대해 fuzzed input을 생성시켜 쓰고 읽는 demo다. 책에서는 bc calculator program으로 진행한다.  

2. Long-Running Fuzzing  
프로그램에 수 많은 입력값을 줘서 그 중 몇이라도 crash가 나는지 testing 해보는 것이 motivation이다. 역시 bc calculator program을 대상으로 테스팅을 하는데 valid arithmetic expression을 가진 fuzzed data가 있다면 stderr가 "" 로 나올 것이다. 100번의 시도중 거의 대부분이 (당연히) invalid 하다. 또한 모든 경우 에러메세지는 "illegal character", "parse error", "syntax error" 중 하나였다. 그리고 returncode가 0이 아닌 경우는 전혀 없었다. 하지만 100번이 아니라 이보다 훨씬 오랫동안 fuzzing을 한다면 결과는 달라질 수 있다. (버그를 찾을 수도 있다)  

### Bugs Fuzzers Find  
Fuzzing의 장점은 많은 일반적이지 않은 입력을 통해 온갖 흥미로운 behavior를 검사할 수 있다는 점이다.  

1. Buffer Overflows  
C언어를 비롯한 대부분의 언어는 buffer overflow에 대해서 error message를 띄우지 않는다. 대신 예측불가능한 행동을 하고 프로그램은 계속 오류가 남아있는 채 돌고 있는 것이다. 여러 케이스의 fuzzed data를 주어 이러한 버그를 잡을 수 있다.  

2. Missing Error Chekcs  
어떤 data에 있어야 할 문자나 숫자가 빠져있는 경우 또한 fuzzing을 통해 testing 가능하다.  

3. Rogue Numbers  
memory 영역을 벗어나는 할당의 경우 시스템의 속도가 현저히 느려지고 통제불능이 되어버려 재부팅밖에는 해결책이 없을 수 있다. 정수에 들어갈 수 없는 큰 수를 생성시켜서 이와 같은 상황에 대해 testing이 가능하다.

### Catching Errors  


### 배운 점  


### 생각 및 질문  
- subprocess module에 대해 공부해보즈아...
