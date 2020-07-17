---
layout: post
title: "프로그래머스 : 땅따먹기"
tags: [알고리즘, 프로그래머스, DP]
comments: true
---

> 2017 카카오코드 본선문제  

[땅따먹기](https://programmers.co.kr/learn/courses/30/lessons/12913)  

### 접근  
전형적인 DP문제이다. 배열의 가장 아래 행부터 바로 윗행에 대해 누적 가능한 선택지에 대해 값을 더해본다. 누적한 값이 기존의 값보다 크다면 DP배열을 업데이트한다. 아래의 경우를 예시로 든다면 4를 선택한 경우, 바로 윗행의 선택할 수 있는 열들의 값(6, 7, 8)에 4를 더하고 그 더한 값이 기존의 값보다 크다면 업데이트한다. 동일한 과정을 3, 2, 1에 대해서도 수행하면 2행의 수들은 각각의 위치에서 3행의 수를 최대값으로 선택한 경우가 된다.  
~~~
1 2 3 5 -> 13 14  15  16
5 6 7 8 -> 8  10  11  12
4 3 2 1
~~~

### 코드  
~~~c++
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int solution(vector<vector<int> > land)
{
    int answer = 0;
    int i, j, k;
    int row = land.size();
    int dp[row][4];

    for (i = 0; i < row; i++) for (j = 0; j < 4; j++) dp[i][j] = land[i][j];

    for (i = row - 1; i > 0; i--) {
        for (j = 0; j < 4; j++) {
            for (k = 0; k < 4; k++) {
                if (k != j) {
                    dp[i-1][k] = max(land[i-1][k] + dp[i][j], dp[i-1][k]);
                    answer = max(answer, dp[i-1][k]);
                }
            }
        }
    }

    return answer;
}
~~~
