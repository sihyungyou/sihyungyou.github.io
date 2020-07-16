---
layout: post
title: "프로그래머스 : 가장 큰 정사각형 찾기"
tags: [알고리즘, 프로그래머스, DP]
comments: true
---

> Programmers  

[가장 큰 정사각형 찾기](https://programmers.co.kr/learn/courses/30/lessons/12905)  

### 접근  
[백준 1915번 : 가장 큰 정사각형](https://www.acmicpc.net/problem/1915)와 동일한 문제다. 풀이는 [이전 포스팅](https://sihyungyou.github.io/baekjoon-1915/)으로 대신한다.  

### 코드  
~~~c++
#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>

using namespace std;

int solution(vector<vector<int>> board)
{
    int answer = 0;
    int i, j;
    int row = board.size();
    int col = board[0].size();
    int map[row+1][col+1];

    for (i = 0; i <= row; i++) {
        for (j = 0; j <= col; j++) {
            if (i == 0 || j == 0) map[i][j] = 0;
            else map[i][j] = board[i-1][j-1];
        }
    }

    for (i = 1; i <= row; i++) {
        for (j = 1; j <= col; j++) {
            if (map[i][j] != 0) {
                int up = map[i-1][j];
                int left = map[i][j-1];
                int diag = map[i-1][j-1];

                if (up != 0 && left != 0 && diag != 0) {
                    int w = min(up, left);
                    w = min(w, diag);
                    w = sqrt(w);
                    map[i][j] = (w + 1) * (w + 1);
                }
            }
            answer = max(answer, map[i][j]);
        }
    }

    return answer;
}
~~~
