---
layout: post
title: "프로그래머스 : 더 맵게"
tags: [알고리즘, 프로그래머스, Heap]
comments: true
---

> Programmers  

[더 맵게](https://programmers.co.kr/learn/courses/30/lessons/42626)  

### 접근  
효율성검사가 있으므로 매번 탐색을 통해 가장 작은 수와 두번째로 작은 수를 찾아내는 것이 아니라 heap 자료구조를 사용해야 한다. 이것에 더하여 고려할 점은 어느 음식도 K 이상으로 만들지 못하는 경우를 처리해주는 것이다. 계속해서 음식을 섞다가 heap에 하나만 남았다고 하자. 만약 마지막 하나 남은 음식의 스코빌 지수가 K 미만이라면 모든 음식을 아무리 섞어도 K 이상으로 만들지 못하는 경우다.  

### 코드  
~~~c++
#include <string>
#include <vector>
#include <queue>

using namespace std;

int solution(vector<int> scoville, int K) {
    int answer = 0;
    int i = 0;
    int len = scoville.size();
    int least = 0;
    int second_least = 0;
    int new_spicy = 0;

    priority_queue <int, vector<int>, greater<int> > pq;
    for (i = 0; i < len; i++) pq.push(scoville[i]);

    while(!pq.empty()) {
        if (!pq.empty()) {
          least = pq.top();
          if (least < K) {
              pq.pop();
              if (!pq.empty()) {
                second_least = pq.top();
                pq.pop();
                new_spicy = least + (second_least * 2);
                pq.push(new_spicy);
                answer++;
              }
              else {
                answer = -1;
                break;
              }
          } else {
              break;
          }
        }
    }

    return answer;
}
~~~
