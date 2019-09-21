---
layout: post
title: "Software Testing Study CH 2-3"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Mutation-Based Fuzzing  

지금까지의 basic fuzzing 과정은 단순히 random string generator 라고 봐도 무방했다. 하지만 이런 전략으로는 문법적으로 invalid input이 입력될 가능성이 매우 높다. 단순히 실행시키는 것보다 실제로 funcionality의 증가를 위해서는 valid input에 대해서 variation을 두는 것이 효과적일 것이다. 이를 성취하기 위한 하나의 방법이 `Mutation-Based Fuzzing`로써 이미 존재하는 valid input의 구조를 조금씩 바꾸는 방법이다.  

예를 들어 Web url parser를 fuzzing 한다고 생각해보자. url은 정해진 문법이 있고 그것을 벗어난 경우에는 유의미한 테스트 케이스로 인정될 수 없다.  
`ParseResult(scheme='http', netloc='www.google.com', path='/search', params='', query='q=fuzzing', fragment='')` 이렇게 specific한 parsing standard/format이 있다는 뜻이다.  
하지만 random generated string이 위와 같은 format의 조건을 만족하기란 거의 불가능에 가깝다. 가능성으로 따지자면 테스팅이 가능한 하나의 valid input을 걸러내기까지 138년의 computing time이 걸릴 정도다. 이러면 software testing의 의미가 전혀 없을 것이다.  

### Mutating Inputs
그래서 단순 fuzzing이 아니라, `Mutating Inputs` 이라는 개념을 더한다.  
random generated string의 대안으로써 하나의 given valid input으로 시작해서 조금씩 바꿔나가는 것이다. 이는 `simple string manipulation` 이라고도 한다. "바꾼다"는 말은 insertion, deletion, flipping a bit in a character 등을 포함한다. 이렇게 하면 앞서 말한 138년이라는 비현실적인 시간동안 valid input을 기다리는 불상사가 생기지 않고 비로소 유의미한 software testing이 가능해진다. 어떤 문자를 delete, insert, flip 할지도 random으로 고르지만 더하여 어떤 방식으로 fuzzing할 것인지도 random 하게 고를 수 있다.  

### Mutating URLs  
위에서 언급한 mutating 방식을 url parser 예제에 적용시켜보자. 먼저 input url에 대해 validity check를 한다. 파이썬의 http_program()가 문자열을 받아들이면 valid하다고 판단한다. 

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
...
~~~

이렇게 더 많은 mutating input이 있을수록 더 다양한 input을 생성할 수 있다. 위의 결과를 살펴보면 original seed input의 형태는 점점 사라져간다. 
이런거를 패키지화 해놓은 것이 MutationFuzzer Class 이다. 이 클래스는 seed, minimum, maximum number of mutation을 인자로 전달받아 fuzzing을 수행한다.  
하지만 the higher variety, the higher risk to having invalid input 라는 것을 명심하자. 단순히 mutation을 많이 한다고 좋은 것만은 아닐 것이다.

### MutationFuzzer Class  
이런 여러 mutations 생성을 single package로 하기 위해서 MutationFuzzer Class를 정의한다. 이 클래스는 seed(list of strings), mutation 최소횟수, 최대횟수를 전달받는다.  

기능으로는 당연히 실제 mutate() 메소드가 있어야 할 것이다.  
create_candidate() 메소드는 diversity in coverage를 최대화 시키기 위한 기능으로, current population에서 랜덤하게 input을 골라서 min~max 에서 랜덤으로 고른 횟수만큼 mutate()을 적용시킨다. 즉 mutation 될 대상을 생성하는 것이다.  
fuzz()는 초기 seed를 고른다. 만약 seed length 보다 seed_index가 적으면 계속 seeding 하고 그렇지 않다면 create candidate한다.  

### Guiding by Coverage  
위에 언급한 문제점을 보완할 수 있는 주요 아이디어가 이런 mutations들을 coverage를 통해 guide 하는 것이다. Coverage는 테스트 퀄리티를 알려주는 좋은 지표이다. specification of program behavior은 있으면 좋겠지만 일단 없다고 가정한다.  
(이 아이디어를 성공적으로 적용한 것이 AFL이다. AFL은 successful한 test case를 evolve하는 방식으로 fuzzing을 진행한다. 이 때 success란 프로그램 실행을 통해 새로운 path를 찾은 test case를 의미한다. 이런 방식을 이용해 AFL은 새로운 path를 찾으면서 mutating을 할 수 있다)   

basic fuzzing과 mutation-based fuzzing 의 testig effectiveness 비교를 위해 coverage information을 수집한다.  
가능한 많은 기능들에 대해서 테스팅을 할 때 까지 발견되는 프로그램 코드를 population에 넣는다. 즉, uncovered code에서 coverage information을 통해 새로운 길을 발견하면 cover하고 (mutation-based fuzzed input으로 테스팅을 하고) 다음 단계로 넘어간다.  

### 배운 점  
- completely random string generating은 validity check에 실패할 확률이 높아 유의미하지 않다.  
- existing valid input 으로부터 mutation을 하면 test quality를 끌어올릴 수 있다.  
