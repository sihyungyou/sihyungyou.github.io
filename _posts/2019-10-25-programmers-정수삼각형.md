---
layout: post
title: "프로그래머스 고득점 kit : 정수 삼각형"
tags: [알고리즘, 프로그래머스, DP]
comments: true
---

> Programmers  

### 문제설명  
[백준 1932번](https://www.acmicpc.net/problem/1932)문제와 같은 문제이다.  

### 접근  
입력의 형태가 약간 달라서 다시 풀어봤다. 풀이는 [여기](https://sihyungyou.github.io/baekjoon-1932/)  

### 코드  
~~~c++
#include <vector>
#include <cstdio>

using namespace std;

int get_max(int a, int b) { return a > b ? a : b; }

int solution(vector<vector<int>> triangle) {
    long long answer = -1;
    int i, j, len;
    len = triangle.size();
    
    for (i = 1; i < len; i++) {
        for (j = 0; j <= i; j++) {
            if (j == 0) triangle[i][j] += triangle[i-1][j];
            else if (j == i) triangle[i][j] += triangle[i-1][j-1];
            else triangle[i][j] = get_max(triangle[i][j] + triangle[i-1][j-1], triangle[i][j] + triangle[i-1][j]);
            
            answer = get_max(answer, triangle[i][j]);
        }
    }
    
    
    return answer;
}
~~~