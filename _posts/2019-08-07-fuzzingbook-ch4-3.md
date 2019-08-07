---
layout: post
title: "Software Testing Study CH 4-3"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Concolic Fuzzing  

이전 장에서 단순히 program crash에만 의존하는 것이 아닌 dynamic taint를 통해 더 좋은(intelligent) 테스트 케이스를 만들어 내는 과정을 보았다. 또한, taint를 이용해 어떻게 문법을 업데이트하고 위험한 함수(method)에 더 집중할 수 있는지도 공부했다.  

이렇듯 taint는 유용하지만 uninterpreted string은 노출될 수 있는 여러 위험요소 중 하나일 뿐이다. 함수가 정확한 길이의 buffer를 항상 인자로 전달받는 것을 확신할 수도 없다. Concolic execution은 이런 문제를 해결해준다.  

Concolic execution의 아이디어는 다음과 같다 : sample input으로 trace 정보를 수집하며 함수를 실행시킨다. 함수가 조건문을 실행하는 지점마다 symbolic variable간의 관계의 형태로 조건들을 저장한다. (이 때 symbolic variable이란 진짜 변수의 placeholder라고 생각하면 편하다. 예를 들어 방정식의 x 처럼 말이다. 그 관계를 설정하는것만으로도 사용할 수 있다.)  

Concolic execution으로 execution path가 만나는 constraint들을 모두 수집할 수 있고 그것으로 program execution path의 특정 지점에서 behavior가 어떤지 알 수 있다. 더하여 concolic execution을 fuzzing에도 사용할 수 있다.  

이번 장에서는 어떻게 Python function을 어떻게 concolically 실행시키는지, 그리고 concolic execution을 어떻게 fuzzing에 접목시키는지 공부한다.  

### Tracking Constraints  
[chapter on information flow](https://sihyungyou.github.io/fuzzingbook-ch4-2/) 에서 어떻게 dynamic taints가 입력으로 인해 닿는 프로그램의 지점들을 이용해서 fuzzing을 유도하는지 배웠다. 하지만 dynamic taint tracking은 그것이 전달할 수 있는 정보에 갇혀있다. 팩토리얼 함수로 예를 들어 보자.  

~~~python
def factorial(n):
    if n < 0:
        return None
    if n == 0:
        return 1
    if n == 1:
        return 1
    v = 1
    while n != 0:
        v = v * n
        n = n - 1
    return v
~~~

![Center example image](https://user-images.githubusercontent.com/35067611/62604162-6aa1c200-b932-11e9-97f5-20db52385cff.png "Center"){: .center-image}  

path [1, 2, 4, 6, 8, 9, 11, 12]는 커버되었지만 [2, 3], [4, 5], [6, 7] 같은 sub-path들은 그렇지 않은 것을 볼 수 있다. 그림에서 보면 2번 조건에서 True로 갈 수 있는 경우의 input을 생성할 필요가 있는 것이다. 그걸 어떻게 할 수 있을까?  

### Concolic Execution  
간단하게 생각해보자. taken path정보를 보고 그 길을 가는 동안 만난 constraint 정보를 모은다. 그리고 non=traversed path로 갈 수 있는 input을 만든다. 예를 들어 위의 그림처럼 특정 branch에서 true 혹은 false (우리가 원하는) 값을 가질 수 있도록 하는 것이다.  
그러기 위한 한 가지 방법이 symbolic variable을 사용하는 것이다. 이는 input을 나타내고, constraint를 암호화하며 SMT solver를 이용해 constraint의 negation을 푼다. 이런 과정에서 만들어질 input이 어떤 조건에 대해 obey해야 하는지 정의하고 그런 조건들을 만족하는 input을 생성하는 것이다.  

### SMT Solvers  
constraint를 풀기 위해서 Satisfiability Modulo Theories (SMT) solver를 사용한다. SMT solver는 SATISFIABILITY (SAT) solver를 기반으로 만들어졌다. SAT solver는 boolean formula를 체크하는데 사용된다. 예를 들어 (a | b ) & (~a | ~b) 는 a = true, b = false 일 때 어떤 경우에도 만족한다. SMT solver는 이런 SAT solver를 확장시켜 string, integer theories를 이해하는 solver로 만들어졌다. 예를 들어 h + t == 'hello,world' 라는 constraint의 경우 SMT solver는 h = 'hello,', t = 'world' 라는 답을 내놓을 수 있다는 것이다.  

위의 팩토리얼 함수에서 n을 입력으로 받았다. 이것을 이용해서 SMT solver가 constraint를 어떻게 해결하는지 예시를 보며 이해해보자.  
~~~python
zn = z3.Int('n')

zn < 0
z3.Not(zn < 0)
z3.solve(z3.Not(zn < 0))
~~~
~~~
n < 0
Not(n < 0)
[n = 0]
~~~

~~~python
x = z3.Real('x')
eqn = (2 * x**2 - 11 * x + 5 == 0)
z3.solve(eqn)
~~~
~~~
[x = 5]
~~~
물론 위의 SMT solver의 답은 여러 답 중 하나이다.  

~~~python
z3.solve(zn < 0)
~~~
~~~
[n = -1]
~~~
이렇듯 SMT solver의 답(-1)을 팩토리얼 함수에 적용시켜보면 실제로 더 많은 coverage를 얻는다.  
zn < 0 뿐만 아니라 다른 uncovered path를 커버하기 위한 여러 constraint에 대해서도 SMT solver에 조건을 넣고 여러 concrete value로 함수를 테스팅해볼 수 있다. concrete value를 사용하면서 동시에 constraint를 저장하기 위해 symbolic shadow variabe를 유지하는 것이다.  

### A Concolic Tracer  
실제 실행된 프로그램의 symbolic context가 있다고 가정하자. 