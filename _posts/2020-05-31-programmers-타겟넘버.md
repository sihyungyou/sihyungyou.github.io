---
layout: post
title: "프로그래머스 고득점 kit : 타겟넘버"
tags: [알고리즘, 프로그래머스, Brute-force, DFS]
comments: true
---

> Programmers  

### 문제설명  
n개의 음이 아닌 정수가 있습니다. 이 수를 적절히 더하거나 빼서 타겟 넘버를 만들려고 합니다. 예를 들어 [1, 1, 1, 1, 1]로 숫자 3을 만들려면 다음 다섯 방법을 쓸 수 있습니다.  
~~~
-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3
~~~
사용할 수 있는 숫자가 담긴 배열 numbers, 타겟 넘버 target이 매개변수로 주어질 때 숫자를 적절히 더하고 빼서 타겟 넘버를 만드는 방법의 수를 return 하도록 solution 함수를 작성해주세요.  

제한사항  
주어지는 숫자의 개수는 2개 이상 20개 이하입니다.  
각 숫자는 1 이상 50 이하인 자연수입니다.  
타겟 넘버는 1 이상 1000 이하인 자연수입니다.  

### 접근  
숫자의 위치는 고정되어 있으며 각 숫자 전에 올 수 있는 연산은 +, - 뿐이다. 숫자 하나에 대해 연산을 선택할 때마다 새로운 경우의 수가 생긴다. 각 경우의 수는 다시 두 갈래로 나뉘어 질 수 있으므로 재귀함수로 구현할 수 있다.  

### 코드  
~~~c++
#include <string>
#include <vector>
#include <cstdio>

using namespace std;
int answer = 0;
int n = 0;
int tar;
vector<int> num;

void dfs(int x, int idx) {
    if (x == tar && idx == n) {
        answer++;
        return;
    } else if (x != tar && idx == n) {
        return;
    } else {
        dfs(x + num[idx], idx + 1);
        dfs(x - num[idx], idx + 1);
    }
}

int solution(vector<int> numbers, int target) {
    tar = target;
    n = numbers.size();
    for(int i = 0; i < n; i++) num.push_back(numbers[i]);

    dfs(0 + num[0], 1);
    dfs(0 - num[0], 1);

    return answer;
}
~~~