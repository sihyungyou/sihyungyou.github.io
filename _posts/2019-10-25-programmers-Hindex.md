---
layout: post
title: "프로그래머스 고득점 kit : H-Index"
tags: [알고리즘, 프로그래머스, 구현]
comments: true
---

> Programmers  

### 문제설명  
H-Index는 과학자의 생산성과 영향력을 나타내는 지표입니다. 어느 과학자의 H-Index를 나타내는 값인 h를 구하려고 합니다. 위키백과1에 따르면, H-Index는 다음과 같이 구합니다.  

어떤 과학자가 발표한 논문 n편 중, h번 이상 인용된 논문이 h편 이상이고 나머지 논문이 h번 이하 인용되었다면 h가 이 과학자의 H-Index입니다.  

어떤 과학자가 발표한 논문의 인용 횟수를 담은 배열 citations가 매개변수로 주어질 때, 이 과학자의 H-Index를 return 하도록 solution 함수를 작성해주세요.  

제한사항  
- 과학자가 발표한 논문의 수는 1편 이상 1,000편 이하입니다.  
- 논문별 인용 횟수는 0회 이상 10,000회 이하입니다.  

입출력 예  
~~~
citations       return
[3, 0, 6, 1, 5] 3
~~~

입출력 예 설명  
이 과학자가 발표한 논문의 수는 5편이고, 그중 3편의 논문은 3회 이상 인용되었습니다. 그리고 나머지 2편의 논문은 3회 이하 인용되었기 때문에 이 과학자의 H-Index는 3입니다.  

### 접근  
문제를 이해하는데 한참 걸렸다. 질문하기란을 보니 다른사람들도 문제가 원하는 답에 대해 상당히 혼란스러워 했던 것 같다. 아무튼 이 문제에서 헷갈릴만한 요소는 [20, 19, 18, 1]이 주어졌을 때 답이 1이아니라 3이라는 것이다.  

1번 이상 인용된 논문이 4편이고 18번 이상 인용된 논문은 18편이 안되므로 답이 1이라고 생각했을 것이다. 하지만 인용횟수는 배열의 값과 별개로 증가해야한다. 18, 19, 20 3편 모두 3번이상 인용되었으므로 답은 3이다. 만약 [20,19,18,17]이었으면 4번이상 인용된 논문이 4편이상이므로 답은 4였을 것이다. 여기서 인용횟수 ret는 더 이상 증가할 필요가 없다.  

문제파악을 했으면 구현은 매우 쉽다. 배열의 값이 검사하는 ret보다 크다면 cnt변수를 증가시킨다. 그리고 그 cnt변수가 ret보다 많고 나머지논문이 ret보다 적은 두 조건을 모두 만족하면 기존의 답과 비교해서 최댓값을 취한다.   

### 코드  
~~~c++
#include <vector>
#include <cstdio>

using namespace std;

int get_max(int a, int b) { return a > b ? a : b; }

int solution(vector<int> citations) {
    int i, cnt, ret, answer = -1;
    int len = citations.size();
    
    ret = 0;
    while(ret <= len) {
        cnt = 0;
        for (i = 0; i < len; i++) if (citations[i] >= ret) cnt++;
        if (cnt >= ret && len-cnt <= ret) answer = get_max(answer, ret);
        ret++;
    }
    
    return answer;
}
~~~