---
layout: post
title: "프로그래머스 : 조이스틱"
tags: [알고리즘, 프로그래머스, Greedy]
comments: true
---

> Programmers  

[조이스틱](https://programmers.co.kr/learn/courses/30/lessons/42860)  

### 접근  
조이스틱의 움직임을 좌우/상하로 나누어서 생각해서 풀었다. 먼저 이전/다음 알파벳으로 바꾸는 상하조작은 각 자리마다 A부터 완성해야하는 이름의 알파벳으로 바꾸는데 몇 번의 조작이 필요한지 계산하고 단순히 더하기만 하면 된다. 좌우의 경우 구현에 신경을 더 써야하는데 기본적인 로직은 현재위치로부터 좌, 우에 바꾸어야 할 알파벳이 더 가까이 있는 쪽으로 이동하는 것이다.  

오른쪽에 바꾸어야 할 알파벳이 있는 거리를 구하는 것은 단순히 현재위치부터 끝까지 탐색하면 된다. 왼쪽 거리 계산은 현재위치보다 왼쪽에 알파벳을 모두 바꿔서 오른쪽 끝으로 돌아가야하는 경우와 왼쪽에 바꿔야할 알파벳이 있는 두 가지로 나누어 생각하면 된다.  

### 코드  
~~~c++
#include <string>
#include <vector>
#include <iostream>

using namespace std;

int solution(string name) {
    int answer = 0;
    int len = name.length();
    int num[len];
    int i, j, left, right, lidx, ridx, change = 0;

    for (i = 0; i < len; i++) {
        char target = name[i];
        num[i] = min(target - 'A', 'Z' - target + 1);
        if (num[i] != 0) change++;
        answer += num[i];
    }

    i = 0;
    int pos = 0;

    if (num[i] != 0) {
        num[i] = 0;
        change--;
    }

    while(change > 0) {

        // 오른쪽 계산
        for (i = pos+1; i < len; i++) {
            if (num[i] != 0) {
                break;
            }
        }

        right = i - pos;
        ridx = i;

        // 왼쪽 계산
        for (i = pos-1; i >= 0; i--) {
            if (num[i] != 0) break;
        }
        if (i < 0) {
            // 왼쪽을 모두 바꿔서 오른쪽 끝으로 돌아가야 하는 경우
            for (j = len-1; j > pos; j--) if (num[j] != 0) break;
            left = pos + (len - j);
            lidx = j;
        } else {
            // 왼쪽에 바꿔야 할 알파벳이 있는 경우
            left = pos - i;
            lidx = i;
        }

        // 이동
        if (left > right) {
            // go right
            answer += right;
            num[ridx] = 0;
            pos = ridx;
        }
        else {
            // go left
            answer += left;
            num[lidx] = 0;
            pos = lidx;
        }

        change--;
    }

    return answer;
}
~~~
