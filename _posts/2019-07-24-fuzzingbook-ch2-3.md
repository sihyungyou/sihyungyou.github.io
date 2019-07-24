---
layout: post
title: "Software Testing Study CH 2-3"
tags: [Software Testing, Fuzzing, 공부]
comments: true
---

> Mutation-Based Fuzzing  

지금까지의 basic fuzzing 과정은 단순히 random string generator 라고 봐도 무방했다. 하지만 이런 전략으로는 문법적으로 invalid input이 입력될 가능성이 매우 높다. 단순히 실행시키는 것보다 실제로 funcionality의 증가를 위해서는 valid input에 대해서 variation을 두는 것이 효과적일 것이다. 이를 성취하기 위한 하나의 방법이 `Mutation-Based Fuzzing`로써 이미 존재하는 valid input의 구조를 조금씩 바꾸는 방법이다.  

예를 들어 Web url parser를 fuzzing 한다고 생각해보자. url은 정해진 문법이 있고 그것을 벗어난 경우에는 유의미한 테스트 케이스로 인정될 수 없다.  
`ParseResult(scheme='http', netloc='www.google.com', path='/search', params='', query='q=fuzzing', fragment='')` 이렇게 specific한 parsing standard/format이 있다는 뜻이다.  
하지만 random generated string이 위와 같은 format의 조건을 만족하기란 거의 불가능에 가깝다. 가능성으로 따지자면 테스팅이 가능한 하나의 valid input을 걸러내기까지 138년의 computing time이 걸릴 정도다. 이러면 software testing의 의미가 전혀 없을 것이다.  

### Mutating Inputs
그래서 단순 fuzzing이 아니라, `Mutating Inputs` 이라는 개념을 더한다.  
random generated string의 대안으로써 하나의 given valid input으로 시작해서 조금씩 바꿔나가는 것이다. 이는 `simple string manipulation` 이라고도 한다. "바꾼다"는 말은 insertion, deletion, flipping a bit in a character 등을 포함한다. 이렇게 하면 앞서 말한 138년이라는 비현실적인 시간동안 valid input을 기다리는 불상사가 생기지 않고 비로소 유의미한 software testing이 가능해진다.  


### Multiple Mutations  
지금까지는 sample string에 한번의 mutation을 적용시켰다. 하지만 여러번의 mutation을 적용시켜 더 많은 변화를 기대할 수 있다. 아래 코드는 10번의 mutation을 실행하는 코드이다.  

~~~python
seed_input = "http://www.google.com/search?q=fuzzing"
mutations = 50

inp = seed_input
for i in range(mutations):
    if i % 5 == 0:
        print(i, "mutations:", repr(inp))
    inp = mutate(inp)
~~~

결과  
~~~
0 mutations: 'http://www.google.com/search?q=fuzzing'
5 mutations: 'http:/L/www.googlej.com/seaRchq=fuz:ing'
10 mutations: 'http:/L/www.ggoWglej.com/seaRchqfu:in'
15 mutations: 'http:/L/wwggoWglej.com/seaR3hqf,u:in'
20 mutations: 'htt://wwggoVgle"j.som/seaR3hqf,u:in'
25 mutations: 'htt://fwggoVgle"j.som/eaRd3hqf,u^:in'
30 mutations: 'htv://>fwggoVgle"j.qom/ea0Rd3hqf,u^:i'
35 mutations: 'htv://>fwggozVle"Bj.qom/eapRd[3hqf,u^:i'
40 mutations: 'htv://>fwgeo6zTle"Bj.\'qom/eapRd[3hqf,tu^:i'
45 mutations: 'htv://>fwgeo]6zTle"BjM.\'qom/eaR[3hqf,tu^:i'
~~~

더 많은 mutating input이 있을수록 더 다양한 input을 생성할 수 있다.  
이런거를 패키지화 해놓은 것이 MutationFuzzer Class 이다. 이 클래스는 seed, minimum, maximum number of mutation을 인자로 전달받아 fuzzing을 수행한다.  
하지만 the higher variety, the higher risk to having invalid input 라는 것을 명심하자. 단순히 mutation을 많이 한다고 좋은 것만은 아닐 것이다.

이런 문제점을 보완할 수 있는 주요 아이디어가 이런 mutations들을 guide 하는 것이다. MutationCoverageFuzzer로 확장시켜 보자.  

### Guiding by Coverage  
먼저 specification of program behavior가 없다고 가정한다. 