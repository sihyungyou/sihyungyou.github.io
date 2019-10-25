---
layout: post
title: "프로그래머스 고득점 kit : 모의고사"
tags: [알고리즘, 프로그래머스, 완전탐색]
comments: true
---

> Programmers  

### 문제설명  
수포자는 수학을 포기한 사람의 준말입니다. 수포자 삼인방은 모의고사에 수학 문제를 전부 찍으려 합니다. 수포자는 1번 문제부터 마지막 문제까지 다음과 같이 찍습니다.  

1번 수포자가 찍는 방식: 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, ...  
2번 수포자가 찍는 방식: 2, 1, 2, 3, 2, 4, 2, 5, 2, 1, 2, 3, 2, 4, 2, 5, ...  
3번 수포자가 찍는 방식: 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, ...  

1번 문제부터 마지막 문제까지의 정답이 순서대로 들은 배열 answers가 주어졌을 때, 가장 많은 문제를 맞힌 사람이 누구인지 배열에 담아 return 하도록 solution 함수를 작성해주세요.  

제한 조건  
- 시험은 최대 10,000 문제로 구성되어있습니다.  
- 문제의 정답은 1, 2, 3, 4, 5중 하나입니다.  
- 가장 높은 점수를 받은 사람이 여럿일 경우, return하는 값을 오름차순 정렬해주세요.  

입출력 예  
~~~
answers     return
[1,2,3,4,5] [1]
[1,3,2,4,2] [1,2,3]
~~~

### 접근  
단순한 brute-force 문제다. 각 수포자의 패턴을 배열에 넣어놓고 정답과 비교하여 맞춘 수를 기록한다.  

### 코드  
~~~c++
#include <vector>
#include <array>
#include <iostream>

using namespace std;
int get_max(int a, int b) { return a > b ? a : b; }
vector<int> solution(vector<int> answers) {
    vector<int> answer;
    array<int, 5> arr1 = { 1,2,3,4,5 };
    array<int, 8> arr2 = { 2,1,2,3,2,4,2,5 };
    array<int, 10> arr3 = { 3,3,1,1,2,2,4,4,5,5 };
    
    int i, j, k, m, max, cnt1 = 0, cnt2 = 0, cnt3 = 0;
    
    j = 0, k = 0, m = 0;
    
    for (i = 0; i < answers.size(); i++) {
        if (j == arr1.size()) j = 0;
        if (k == arr2.size()) k = 0;
        if (m == arr3.size()) m = 0;
        
        if (arr1[j] == answers[i]) cnt1++;
        if (arr2[k] == answers[i]) cnt2++;
        if (arr3[m] == answers[i]) cnt3++;
        
        j++; k++; m++;
    }
    
    max = get_max(cnt1, cnt2);
    max = get_max(max, cnt3);
    
    if (max == cnt1) answer.push_back(1);
    if (max == cnt2) answer.push_back(2);
    if (max == cnt3) answer.push_back(3);
    
    return answer;
}
~~~