---
layout: post
title: "Software Testing Study CH 2-1"
tags: [Software Testing, Fuzzing, 공부]
comments: true
---

> Fuzzing: Breaking Things with Random Inputs  

둘째 장의 첫 토픽은 `Fuzzing`에 대한 간략한 소개와 배경설명이다. `Fuzzing` 이란 무엇인며 어떻게 내부적으로 구현되어있는지 정리한다. 또한 이를 이용해서 `Software Testing`에 어떻게 적용하는지 공부해보았다.  

### What is Fuzzing?  
invalid, unexpected, random data 를 입력으로 받아 버그 테스팅을 하는 기술을 말한다. 비가 많이 오고 천둥이 치던 밤 우연히 발견되었는데 뇌우가 컴퓨터를 연결해놓은 전화선에 노이즈를 발생시켰고 이것이 UNIX command를 crash 하거나 bad input을 집어넣었다. 프로그램이 그런 random input에 의해 crash될 정도로 견고하지 못하다는 것에 놀란 한 교수는 이 노이즈가 미치는 영향에 대해서 과학적으로 조사하기 시작했고 fuzzing이 발전하게 되었다.  

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
위의 버그들 (hang or crash를 야기하는) 외에도 여러 에러들 또한 잡을 수 있다.  

1. Generic Chekcers  
Generic, 뜻 그대로 일반적인 오류를 점검한다. 특정 프로그램에 국한되는 것이 아니라 개발자라면 누구나 흔히 하는 실수들에 대한 테스팅이다.  
- Checking Memory Access : 할당되지 않은 메모리 영역에 접근하는지 여부를 체크한다. 일반적으로 out of bounds 에러를 의미한다.  
- Information Leaks : 컴퓨터에게 기대하는 reply보다 그 length가 클 경우에 그 뒤에 붙여진 정보들이 누출된다. 이러한 이슈를 잡아내기 위해서 누출되면 안 되는 정보를 marker로 정해놓고 reply에서 marker까지 return했다면 leak이라고 판단한다.  

2. Program-Specific Chekcers  
모든 프로그램에 적용될 수 있는 generic checkers와는 다르게 특정한 프로그램이 요하는 오류에 대해 validity를 테스팅 한다. 주로 assertion 개념을 통해 error detecting을 한다. 프로그램마다 목적과 용도가 다르기 때문에 program-specific checkers는 개발자가 직접 함수로 정의한다.  

3. Static Code Checkers  
assertion 으로 program-specific chekcers을 수행했다면 static type checkers로도 같은 역할을 수행할 수 있다. 예를 들면 파이썬에는 str, str type의 Dict에 대해 int, str이 들어올 경우 error detecting 을 해주는 MyPy static checker가 있다.  

### Fuzzing Architecture  
위와 같은 여러 테스팅을 하기 위해서 fuzzing 기술을 이용한다고 했다. 그렇다면 실제 runner, fuzzer class는 어떻게 구현되어있는지 살펴보자.  

~~~python
class Runner(object):
    # Test outcomes
    PASS = "PASS"
    FAIL = "FAIL"
    UNRESOLVED = "UNRESOLVED"

    def __init__(self):
        """Initialize"""
        pass

    def run(self, inp):
        """Run the runner with the given input"""
        return (inp, Runner.UNRESOLVED)
~~~
run() 함수는 input을 runner에게 전달하고 result, outcome pair를 반환한다. 이 때 result는 runner-specific value이며 outcome은 테스트 통과 여부다.

~~~python
class Fuzzer(object):
    def __init__(self):
        pass

    def fuzz(self):
        """Return fuzz input"""
        return ""

    def run(self, runner=Runner()):
        """Run `runner` with fuzz input"""
        return runner.run(self.fuzz())

    def runs(self, runner=PrintRunner(), trials=10):
        """Run `runner` with fuzz input, `trials` times"""
        # Note: the list comprehension below does not invoke self.run() for subclasses
        # return [self.run(runner) for i in range(trials)]
        outcomes = []
        for i in range(trials):
            outcomes.append(self.run(runner))
        return outcomes
~~~

~~~python

class RandomFuzzer(Fuzzer):
    def __init__(self, min_length=10, max_length=100,
                 char_start=32, char_range=32):
        """Produce strings of `min_length` to `max_length` characters
           in the range [`char_start`, `char_start` + `char_range`]"""
        self.min_length = min_length
        self.max_length = max_length
        self.char_start = char_start
        self.char_range = char_range

    def fuzz(self):
        string_length = random.randrange(self.min_length, self.max_length + 1)
        out = ""
        for i in range(0, string_length):
            out += chr(random.randrange(self.char_start,
                                        self.char_start + self.char_range))
        return out
~~~
Fuzzer의 에서 주목할 것은 fuzz()와 run()이다. 먼저 fuzz()는 입력값을 그대로 반환함으로써 출력하거나 다른 메소드에 전달할 수 있도록 한다. 하지만 아래 코드 RandomFuzzer 에서는 랜덤한 문자열을 생성하는 역할을 한다. run()을 통해 fuzzed data를 Runner class에 전달하여 result, outcome pair를 반환받는다.  


### 배운 점  
- Fuzzer, Runner에 대한 기본 개념, 사용법 및 배경  
- Fuzzing으로 생성한 input은 랜덤하면서 많은 경우의 수를 통해 예상하지 못했던 여러 behavior들을 이끌어냄으로써 Software Testing 영역에 적용된다.  

### 생각 및 질문  
- python subprocess module에 대해 공부해보즈아...  

