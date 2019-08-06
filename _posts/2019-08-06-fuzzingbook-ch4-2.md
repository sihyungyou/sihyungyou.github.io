---
layout: post
title: "Software Testing Study CH 4-2"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Tracking Information Flow

지금까지 어떻게 더 나은(프로그램에 더 깊숙히 관여할 수 있는) input을 생성하는지에 집중했다. 그 과정에서 프로그램에서 어떤 문제를 찾았는지 알기 위해 프로그램이 crash하는 경우에 의존했다. 하지만 이런 접근은 너무 단순하다. 만약 프로그램의 동작 자체가 잘못됐는데 crash하지 않는다면 어떻게 해야할까? 이번 장에서는 information flow를 트래킹하는 것을 공부한다. 그리고 이 flow는 프로그램의 동작이 기대했던 바와 같이 작동하는지 판단할 수 있도록 도와줄 것이다.  

### Vernerable Database  
tracking information flow를 본격적으로 보기 전에 책에서는 먼저 파이썬으로 sql 형식의 database를 다루는 클래스를 하나 만든다. 일반적인 sql문들과 다를 바가 없으므로 구현 부분은 생략한다. 그리고 fuzzing을 한다. 결과는 다음과 같은데 프로그램에 어떤 crash도 일으키지 않는다. 하지만 그 외에 걱정해야할 부분(버그들)이 있을 수 있다.  

~~~
'select O6fo,-977091.1,-36.46 from inventory'
>  Invalid WHERE ('(O6fo,-977091.1,-36.46)')

'select g3 from inventory where -3.0!=V/g/b+Q*M*G'
>  Invalid WHERE ('(-3.0!=V/g/b+Q*M*G)')

'update inventory set z=a,x=F_,Q=K where p(M)<_*S'
>  Column ('z') was not found

'update inventory set R=L5pk where e*l*y-u>K+U(:)'
>  Column ('R') was not found

'select _/d*Q+H/d(k)<t+M-A+P from inventory'
>  Invalid WHERE ('(_/d*Q+H/d(k)<t+M-A+P)')

'select F5 from inventory'
>  Invalid WHERE ('(F5)')

'update inventory set jWh.=a6 where wcY(M)>IB7(i)'
>  Column ('jWh.') was not found

'update inventory set U=y where L(W<c,(U!=W))<V(((q)==m<F),O,l)'
>  Column ('U') was not found

'delete from inventory where M/b-O*h*E<H-W>e(Y)-P'
>  Invalid WHERE ('M/b-O*h*E<H-W>e(Y)-P')

'select ((kP(86)+b*S+J/Z/U+i(U))) from inventory'
>  Invalid WHERE ('(((kP(86)+b*S+J/Z/U+i(U))))')
~~~

### The Evil of Eval  
책에서 구현한 클래스는 Python interpreter를 사용해서 eval() 함수를 사용한다.  

~~~python
db.sql('select year - 1900 if year < 2000 else year - 2000 from inventory')
~~~
~~~
[98, 0, 99]
~~~
yesr - 1900, yearh < 2000, year - 2000 등이 가능한 이유는 valid Python expression이기 때문이다. (물론 SQL expression에서는 그렇지 않다)  

문제는 Python expression의 한계가 없다는 점이다. 다음의 예시를 보자.  

~~~python
db.sql('select __import__("os").popen("pwd").read() from inventory')
~~~
~~~
['/Users/zeller/Projects/fuzzingbook/notebooks\n',
 '/Users/zeller/Projects/fuzzingbook/notebooks\n',
 '/Users/zeller/Projects/fuzzingbook/notebooks\n']
 ~~~

위의 문장은 효과적으로 유저의 파일시스템을 읽어온다. os.popen("pwd").read() 대신에 다른 Python command 또한 얼마든지 실행시킬 수 있다는 뜻이다. 그리고 그 command는 유저의 데이터에 접근하고 소프트웨어를 설치하고 백그라운드 프로세스를 실행시킬 수도 있다. 이것이 "the full power of Python expressions"이다.  

물론 그 full power를 이용하는 것이 목적이다. 단, 믿을 수 없는 유저나 다른 third party는 그래서느 안된다. 그러므로 믿을 수 있는 input과 그렇지 않은 input을 구분할 수 있어야 한다. 그러기 위해서 취할 수 있는 한 가지 방법은 dynamic taint analysis 이다. 이것은 user input 받아들이는 함수들을 identify 하는 것이다.  
1. source functions : those accept user input and taint any string that comes in through them  
2. sink functions : those perform dangerous operations as sinks  
3. taint sanitizer functions  
source function으로부터 나온 입력들은 절대 sanitization function을 거치지 않고서는 sink function로 갈 수 없다. 이런 방식의 분석은 단순히 crash check 뿐만 아니라 stronger oracle로 테스팅을 할 수 있도록 한다.  

