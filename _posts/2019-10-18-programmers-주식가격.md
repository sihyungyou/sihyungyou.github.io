---
layout: post
title: "프로그래머스 고득점 kit : 주식가격"
tags: [알고리즘, 프로그래머스, Queue]
comments: true
---

> Programmers  

### 문제설명  
초 단위로 기록된 주식가격이 담긴 배열 prices가 매개변수로 주어질 때, 가격이 떨어지지 않은 기간은 몇 초인지를 return 하도록 solution 함수를 완성하세요.  

제한사항  
- prices의 각 가격은 1 이상 10,000 이하인 자연수입니다.  
- prices의 길이는 2 이상 100,000 이하입니다.  

입출력 예  
~~~
prices          return
[1, 2, 3, 2, 3] [4, 3, 1, 1, 0]
~~~

입출력 예 설명  
1초 시점의 ₩1은 끝까지 가격이 떨어지지 않았습니다.  
2초 시점의 ₩2은 끝까지 가격이 떨어지지 않았습니다.  
3초 시점의 ₩3은 1초뒤에 가격이 떨어집니다. 따라서 1초간 가격이 떨어지지 않은 것으로 봅니다.  
4초 시점의 ₩2은 1초간 가격이 떨어지지 않았습니다.  
5초 시점의 ₩3은 0초간 가격이 떨어지지 않았습니다.  

### 접근  
큐를 꼭 사용하지 않아도 풀릴 듯 하다. prices 정보를 큐에 넣고 하나씩 빼면서 나머지 값들과 비교하며 초를 증가시켜주고 떨어지는 순간에 반복문을 탈출한다. 1초뒤에 가격이 떨어지는 것도 1초간 떨어지지 않은 것으로 인정하기 떄문에 반복문을 탈출하는 순간에도 초는 증가시켜줘야한다.  

### 코드  
~~~c++
#include <queue>
#include <vector>

using namespace std;

vector<int> solution(vector<int> prices) {
    vector<int> answer;
    queue <int> q;
    int i, j, cur, cnt = 0;
    
    for (i = 0; i < prices.size(); i++) q.push(prices[i]);
    
    i = 0;
    while(!q.empty()) {
        cnt = 0;
        cur = q.front();
        q.pop();
        for (j = i+1; j < prices.size(); j++) {
            if (cur > prices[j]) {
                cnt++;
                break;
            }
            cnt++;
        }
        answer.push_back(cnt);
        i++;
    }
    return answer;
}
~~~