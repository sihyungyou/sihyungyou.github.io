---
layout: post
title: "Software Testing Study CH 1"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Introduction to Software Testing  

### 포스팅 배경  
2019년 가을학기 공학프로젝트기획의 주제로 `Security Bug 검출을 위한 Greybox Fuzz Testing 기술 및 도구 개발`를 선택했다. 뭔가 멋있고 심오한데 일단 주제에 대해서 단 1도 모르는 상태다. 이번 여름부터 책이나 논문 등을 읽으며 background를 쌓고 학기 중에 교수님의 지도에 따라 공부를 계속 할 예정이다. 먼저 방학 중 스터디로 [fuzzingbook](https://www.fuzzingbook.org)의 내용을 공부하게 되었다. 앞으로의 포스팅들은 이 책에서 내가 아는 것, 배운 것, 모르는 것들을 정리하는 포스팅이다. 교재의 programming demo는 웹사이트에서 직접 코드를 고치고 돌려볼 수 있으므로 코드 실습 설명보다는 이론적인 내용을 정리한다.  

### 요약  
첫 장에서는 먼저 `Software Testing`이 무엇인지 간단하게 설명하고 간단한 프로그래밍을 통해 감을 잡는다. 또한 점진적인 접근으로 테스팅이 왜 필요한지, 얼만큼 해야 충분한지, 성공적인 테스팅이라고 어떻게 결정할 수 있는지를 알아본다.  
 
간단한 예제 루트를 구하는 코드로 시작해보자. [Newton-Raphson metohd](https://en.wikipedia.org/wiki/Newton%27s_method)에 따라 코드를 구현하고 이 알고리즘이 올바르게 동작하는지 알아본다.  

~~~python
def my_sqrt(x):
    """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx
~~~

위의 코드에서 값이 올바른지 확인하는 첫 번째 방법은 함수 내에 print log를 찍어보는 것이다. 이 방법을 통해 반복문이 도는 과정에서 값의 변화를 눈으로 확인할 수 있다. 흔한 디버깅 방법이다. 하지만 코드가 돌아가는 데에 에러가 생기지 않았더라도 값이 맞다는 것을 증명하지는 못한다.  

두 번째 방법으로 √× * √× = x 의 성질을 이용해서 실제로 값이 맞는지 확인하는 것이다. 
my_sqrt(2) * my_sqrt(2) 의 결과는 1.9999999999999996 으로 정확히 2는 아니지만 rounding error를 감안해서 본다면 맞는 결과인 것 같다. 대안으로 아주 작은 값을 입실론으로 정하고 결과값과 기댓값의 차이가 입실론보다 작으면 맞는 것으로 간주하는 방법이 있다.

이렇게 하나 하나 테스팅을 하는 것은 testing input이 매우 한정적이고 매번 반복해서 코드를 돌려야하는 한계점이 있기에 위 과정을 함수화 시켜서 automated testing을 해준다.  

그럼에도 여전히 프로그래머가 직접 값을 변경시키는 작업이 반복되어야 하므로 random number generator를 이용해서 많은 test case를 한번에 반복문으로 처리하는 코드로 바꿔준다.  

위의 과정을 통해 testing이 어느정도 되었다고 판단하면 코드를 배포한다. 하지만 에러처리는 되지 않은 상태이다. 그러므로 입력값이 0보다 작은경우, 0인 경우, 숫자가 아닌 경우 등 최대한 많은(?) 에러케이스를 처리하는 코드도 추가해준다.  

에러처리까지 모두 마쳤다면 우리는 my_sqrt 함수가 올바르게 작동한다고 "믿는다". 우리는 최대한 잘 고른, 그리고 많은 입력값으로 테스팅을 해서 정확도를 높인다.  

### 배운 점  
- 지금까지 디버깅의 일환으로 했던 로그 출력조차도 Software Testing이라고 할 수 있다.  
- Sortware Testing이 100% 보장해주지는 않는다.  

### 생각 및 질문  
- Testing의 확률을 높이려면 어떻게 해야 할까  
- 아직 fuzzing이라는 단어조차 안 나왔는데 뭔지 궁금..(?)  
