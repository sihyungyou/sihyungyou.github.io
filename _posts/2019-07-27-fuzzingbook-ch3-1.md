---
layout: post
title: "Software Testing Study CH 3-1"
tags: [Software Testing, Fuzzing, 공부]
comments: true
---

> Fuzzing with Grammars  

CH 2-3에서 test generatio의 속도를 올리고 최대한 valid input 범위 내에서 fuzzing을 할 수있도록 하는 Mutation based fuzzing에 대해 공부했다. 여기서 한 단계 더 발전시켜 프로그램에 specification of legal input을 만들어주려 한다. Fuzzing with grammars를 통해서 하는데 이는 valid input을 생성시킬 뿐 아니라 더 복잡한 입력일수록 효과를 발휘하게 된다.  

프로그램에 대한 입력은 온갖 (실패를 포함해서) program behavior를 야기한다. 그렇기 때문에 입력들을 효과적, 체계적으로 관리하는 것은 매우 중요하다. Grammars는 syntatical structure of input을 설명하는데 매우 좋으며 재귀적으로 그 표현을 간단하게 정리할 수 있다는 장점이 있다.  

### Rules and Expansions  
Grammar는 start symbol로 시작해서 set of expansion으로 그 규칙을 확장시켜 나간다. 예를 들어, 아래는 두 자리 숫자를 나타내는 grammar이다.
~~~
<start> ::= <digit><digit>
<digit> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
~~~

여기서 ::= 표시는 "lhs가 rhs로 대체될 수 있다"를 의미한다. grammar가 재귀적으로 불릴 수 있다는 것은 아래와 같은 경우를 의미한다.  
~~~
<start>   ::= <number>
<number>  ::= <integer> | +<integer> | -<integer>
<integer> ::= <digit> | <digit><integer>
<digit>   ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
~~~

위의 예들과 같은 grammar에서 start로 시작하여 다른 symbol들로 대안들을 찾아나가는 과정을 반복한다면 빠르게 valid expression을 생성시킬 수 있을 것이다. 이 과정이 곧 grammar fuzzing이고 이는 복잡하지만 문법에서 허용되는 입력을 빠르게 생성시키는데 매우 효과적이다.  

