---
layout: post
title: "Fuzzingbook Exercises CH 2-1"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Fuzzing: Breaking Things with Random Inputs  

### Exercise 1: Simulate Troff  
For each of the above, write a Python function `f(s)` that fails if `s` fulfills the failure criterion.  

~~~python
import string

def noabc(inp):
    pattern = "abc"
    index = inp.find(pattern)
    assert index < 0 or index + len(pattern) > len(inp), "err message"
~~~
~~~python
with ExpectError():
    noabc("abcde")
~~~
패턴을 찾지 못하거나 (index == -1), 패턴이 시작하는 index로부터 길이를 더한 값이 len를 넘어가면 입력 문자열에 패턴이 없는 것으로 판단하므로 return True. 하지만 위의 조건을 만족하지 않는다면 에러 메시지를 출력한다.  

~~~python
def no_8bit(inp):
    for i in range(len(inp) - 1):
        assert ord(inp[i]) <= 127 or inp[i + 1] != '\n'
    return True
~~~
~~~python
with ExpectError():
    no_8bit("ä\n")
~~~
입력 문자열의 마지막 전 문자까지 검사하여 ASCII 코드 127번까지로 표현할 수 없다면 에러 메시지 출력.  

~~~python
def no_dot(inp):
    assert inp != ".\n"
    return True
~~~
input string에 dot이 있다면 에러 메시지 출력.  

### Exercise 2: Run Simulated Troff  
Create a class `TroffRunner` as subclass of `Runner` that checks for the above predicates. Run it with `Fuzzer`.  Be sure to have the `Fuzzer` object produce the entire range of characters. Count how frequently the individual predicates fail.  

~~~python
class TroffRunner(Runner):
    def __init__(self):
        self.no_backslash_d_failures = 0
        self.no_8bit_failures = 0
        self.no_dot_failures = 0

    def run(self, inp):
        try:
            no_backslash_d(inp)
        except AssertionError:
            self.no_backslash_d_failures += 1

        try:
            no_8bit(inp)
        except AssertionError:
            self.no_8bit_failures += 1

        try:
            no_dot(inp)
        except:
            self.no_dot_failures += 1

        return inp
~~~
~~~python
random_fuzzer = RandomFuzzer(char_start=0, char_range=256, max_length=10)
troff_runner = TroffRunner()

trials = 100000
for i in range(trials):
    random_fuzzer.run(troff_runner)
~~~
들어올 각각의 input 마다 어떤 종류의 failure 인지 count하기 위해 변수를 선언한다. 그리고 위에서 정의한 함수들을 각 input에 대해 실행시켜보고 에러가 검출돌 경우 counting variable를 올린다.  

### Exercise 3: Run Real Troff  
Using `BinaryProgramRunner`, apply the fuzzer you configured on the real `troff` program.  Check if you can 
produce any run whose output code is non-zero, indicating a failure or a crash.  

~~~python
real_troff_runner = BinaryProgramRunner("troff")
for i in range(100):
    result, outcome = random_fuzzer.run(real_troff_runner)
    if outcome == Runner.FAIL:
        print(result)
~~~