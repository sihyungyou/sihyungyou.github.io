---
layout: post
title: "Software Testing Study CH 2-6"
tags: [Software Testing, Fuzzing, 공부]
comments: true
---

> Mutation Analysis  

Code coverage 챕터에서는 프로그램에서 어떤 부분이 실제로 실행되었는지 확인하는 방법과 그에 기반하여 테스트 케이스들의 유효성을 알아볼 수 있었다. 하지만 coverage 자체로는 테스팅의 효율을 확인하는 데에 불충분한데 coverage가 아무리 높더라도 결과의 정확성을 보장하지 않기 때문이다. 틀린 결과값이 나와도 coverage information 측면에서는 옳은 결과와 똑같이 취급한다는 것이다. 즉, 100% coverage 라도 버그를 찾는 데에는 어떤 일도 하지 않았을 수도 있다.  

따라서 이번 장에서는 테스팅의 유효성을 알아보는 또 다른 방법을 소개하는데 이는 코드에 injecting mutations (artificial faults)을 하는 방법이다. 간단히 말하면, artificial faults를 코드에 주입시킨 후 테스트 케이스가 그것들을 찾아내는지를 확인하는 것이다. 만약 그것들조차 찾지 못한다면 실제 버그도 찾지 못할 가능성이 매우 높다.  

artificial faults는 말 그대로 사람이 직접 만든 인공적인 버그이다. 이런 버그를 만든다는 것이 결코 쉬운 일이 아니다. 그 이유는 먼저 이미 코드를 직접 짠 프로그래머의 preconception 때문이다. 또한, 좋은 버그를 작성하는 것은 시간이 매우 오래걸리기 때문이다.  

### Seeding Artificial Faults with Mutation Analysis  
Mutation Analysis는 테스트 케이스의 유효성을 측정하는 새로운 대안이다. 코드를 약간 수정해서 (예를 들면 +를 -로 바꾸는 등의) 그 버그를 찾는지 보는 것이다. 이런 코드 내에 심어놓은 mutation들을 찾으면 그 test suite는 점수를 얻게 되는데 많이 찾을 수록 그 점수를 많이 받는다. (전체 mutation 중 찾은 artificial fault의 비율로 측정한다)  

### Structural Coverage Adequacy by Example  
triangle 함수를 통해 coverage only가 어떤 점이 부족하고 mutation analysis는 그것을 어떻게 보완하는지 알아보자.  
~~~python
def triangle(a, b, c):
    if a == b:
        if b == c:
            return 'Equilateral'
        else:
            return 'Isosceles'
    else:
        if b == c:
            return "Isosceles"
        else:
            if a == c:
                return "Isosceles"
            else:
                return "Scalene"
~~~

two different test cases :  
~~~ python
def strong_oracle(fn):
    assert fn(1, 1, 1) == 'Equilateral'

    assert fn(1, 2, 1) == 'Isosceles'
    assert fn(2, 2, 1) == 'Isosceles'
    assert fn(1, 2, 2) == 'Isosceles'

    assert fn(1, 2, 3) == 'Scalene'
~~~
~~~python
def weak_oracle(fn):
    assert fn(1, 1, 1) == 'Equilateral'

    assert fn(1, 2, 1) != 'Equilateral'
    assert fn(2, 2, 1) != 'Equilateral'
    assert fn(1, 2, 2) != 'Equilateral'

    assert fn(1, 2, 3) != 'Equilateral'
~~~

하지만 두 test suite에 대해서 coverage information은 동일하다.  
~~~
   1: def triangle(a, b, c):
#  2:     if a == b:
#  3:         if b == c:
#  4:             return 'Equilateral'
   5:         else:
#  6:             return 'Isosceles'
   7:     else:
#  8:         if b == c:
#  9:             return "Isosceles"
  10:         else:
# 11:             if a == c:
# 12:                 return "Isosceles"
  13:             else:
# 14:                 return "Scalene"
  15: 
~~~

### Injecting Artificial Faults  
이전까지와는 다르게 프로그램에 버그를 심고 test suite가 그것들을 detect하는 frequency를 측정하려 한다. 이런 기술은 fault injection이라고 불린다. 아래의 예와 같이 의외로 단순한 개념이다.  
~~~python
def triangle_m1(a, b, c):
    if a == b:
        if b == c:
            return 'Equilateral'
        else:
            # return 'Isosceles'
            return None  # <-- injected fault
    else:
        if b == c:
            return "Isosceles"
        else:
            if a == c:
                return "Isosceles"
            else:
                return "Scalene"
~~~

결과를 보면 위의 fault는 weak_oracle은 잡아내지 못하지만 strong_oracle은 잡아낼 수 있다.  
fault injection은 test suite의 유효성을 잘 측정해낼 수 있다. 하지만 문제는 unbiased fault를 수집하는 것은 꽤나 어렵다는 것이다. resonably hard to detect, manual process, good falut를 만드는 것은 쉽지 않다. 그러므로 지금 단계에서 code coverage보다 낫다고 하기에는 충분치 않다.  

Mutation Analysis는 잘 다듬어진 set of fault의 좋은 대안을 제시한다. 프로그램 내에 valid variants(mutants)를 생성시키는 것이다. test suite가 이런 mutant에 대해 테스팅을 하고 유효성은 detected mutants (killed) / all valid mutants 로 계산한다.  

### Mutating Python Code  
그럼 코드를 어떻게 mutate 시킬까? abstract syntax tree (AST) 개념을 도입해보자. 간단히 말하면 프로그램을 트리형태로 바꾸고 그 트리의 부분들을 바꾸는 것이다. 물론 트리형태는 다시 textual form으로 변환 가능하다.  

![Center example image](https://user-images.githubusercontent.com/35067611/62007570-fa54ad00-b189-11e9-9a6f-5985f799ffaf.png "Center"){: .center-image}  

### A Simple Mutator for Functions  
그렇다면 어떻게 프로그램에서 valid한 mutant를 만들 수 있을까? 가장 간단한 방법은 몇몇 statement를 pass로 대체하는 것이다. (코드상의 구현은 설명이 너무 길어지므로 생략한다)  

### Evaluating Mutations  
~~~python
for mutant in MuFunctionAnalyzer(function):
    with mutant:
        assert function(x) == y
~~~
mutant가 active할 동안 기존의 함수는 mutant 함수로 대체되어 작동한다.  

### The Problem of Equivalent Mutants  
Mutant analysis의 문제점은 생성된 모든 mutants가 fault는 아니라는 것이다. 예를 들어 아래 코드 결과에서 1번 mutant는 기존의 것과 구별이 불가능하다. 이런 것들을 equivalent Mutants라고 하는데 mutantion score을 제대로 계산하기 매우 어려워 진다. 그러므로 equivalent mutants의 갯수를 정확히 파악하고 mutant analysis를 해야한다.  
~~~python
def new_gcd(a, b):
    if a < b:
        a, b = b, a
    else:
        a, b = a, b

    while b != 0:
        a, b = b, a % b
    return a
~~~
아래와 같이 mutation 했을 때,  
~~~python
def gcd(a, b):
    if a < b:
        a, b = b, a
    else:
        pass

    while b != 0:
        a, b = b, a % b
    return a
~~~

결과  
~~~python
for i, mutant in enumerate(MuFunctionAnalyzer(new_gcd)):
    print(i,mutant.src())
~~~
~~~
0 def new_gcd(a, b):
    if a < b:
        pass
    else:
        a, b = a, b
    while b != 0:
        a, b = b, a % b
    return a

1 def new_gcd(a, b):
    if a < b:
        a, b = b, a
    else:
        pass
    while b != 0:
        a, b = b, a % b
    return a

2 def new_gcd(a, b):
    if a < b:
        a, b = b, a
    else:
        a, b = a, b
    while b != 0:
        pass
    return a

3 def new_gcd(a, b):
    if a < b:
        a, b = b, a
    else:
        a, b = a, b
    while b != 0:
        a, b = b, a % b
    pass
~~~