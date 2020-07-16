---
layout: post
title: "프로그래머스 : 위장"
tags: [알고리즘, 프로그래머스, Hash, 수학]
comments: true
---

> Programmers  

[위장](https://programmers.co.kr/learn/courses/30/lessons/42578)  

### 접근  
map을 이용하여 옷의 종류별 개수를 파악하고 조합의 수를 계산해야겠군! 이라는 생각까지는 들었으나.. 어떻게 계산해야할지 수식을 세우지 못했다. 결국 [TaeYang Lim님의 블로그](https://velog.io/@giraffelim/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8%EC%8A%A4-%EC%9C%84%EC%9E%A5)를 참고했다.  

옷의 종류별 개수를 곱하되 입지 않는 경우를 고려하여 +1 해서 곱한다. 그리고 모두 입지 않는 경우는 없으므로 정답에서 1을 뺀 후 return 한다. 이런 수학적 수식을 세우는 것이 여전히 약한 것 같다 ㅠㅠ  

### 코드  
~~~c++
#include <string>
#include <vector>
#include <map>
using namespace std;

int solution(vector<vector<string>> clothes) {
    int answer = 1;
    int i, j, len = clothes.size();
    map <string, int> m;
    map <string, int>::iterator it;

    // 옷의 종류별 개수 파악
    for (i = 0; i < len; i++) {
        string s = clothes[i][1];
        it = m.find(s);
        if (it != m.end()) {
            m[s]++;
        }
        else {
            m[s] = 1;
        }
    }

    // 블로그 참고 부분
    for (it = m.begin(); it != m.end(); it++) {
        answer *= (it.second + 1);
    }

    return answer - 1;
}
~~~
