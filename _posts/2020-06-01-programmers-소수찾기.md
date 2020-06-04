---
layout: post
title: "프로그래머스 고득점 kit : 소수찾기"
tags: [알고리즘, 프로그래머스, Brute-force]
comments: true
---

> Programmers  

### 문제설명  
한자리 숫자가 적힌 종이 조각이 흩어져있습니다. 흩어진 종이 조각을 붙여 소수를 몇 개 만들 수 있는지 알아내려 합니다.  

각 종이 조각에 적힌 숫자가 적힌 문자열 numbers가 주어졌을 때, 종이 조각으로 만들 수 있는 소수가 몇 개인지 return 하도록 solution 함수를 완성해주세요.  

제한사항
numbers는 길이 1 이상 7 이하인 문자열입니다.  
numbers는 0~9까지 숫자만으로 이루어져 있습니다.  
013은 0, 1, 3 숫자가 적힌 종이 조각이 흩어져있다는 의미입니다.  

### 입출력 예시  
~~~
numbers return
17      3
011     2
~~~

### 접근  
에라토스테네스의 체와 nex_permutation 개념을 함께 사용하면 풀 수 있는 문제다. 즉, 들어온 입력으로 만들 수 있는 모든 수열 경우의 수에 대해 소수검사를 하면 된다. 다만 주의해야할 점이 있다. 입력의 길이가 4라면 (ex. 1234) 모든 수열은 1234, 1243, 1324, ... 등 네자리 수로 나온다. 이 네자리 수에 대해서만 소수 검사를 하면 안된다. 예를 들어 1234의 경우 4, 34, 234, 1234 모두에 대해 검사를 해야 한다.  

### 코드  
~~~c++
#include <string>
#include <cmath>
#include <algorithm>
#include <set>
#include <cstdio>
#include <utility>

using namespace std;

int solution(string numbers) {
    int max_num = 9999999;
    int answer = 0;
    int len = numbers.length();
    int i, j;
    bool check[max_num+1];
    int arr[len];
    set<int> s;

    for (i = 0; i <= max_num; i++) {
        check[i] = true;
    }

    check[0] = false;
    check[1] = false;

    // 에라토스테네스의 체
    for (i = 2; i <= sqrt(max_num); i++) {
      if (check[i]) {
        for (j = i + i; j <= max_num; j += i) {
            if (check[j]) check[j] = false;
        }
      }
    }

    // 문자열 -> array로
    for (i = 0; i < len; i++) arr[i] = numbers[i] - '0';

    // sort array
    sort(arr, arr+len);

    // permutation 돌면서 소수인지 확인
    do {
        int temp = 0;
        int ten = 1;
        int start = 0;

        for (i = len-1; i >= 0; i--) {
            pair<set<int>::iterator, bool> pr;
            temp += arr[i] * ten;
            pr = s.insert(temp);
            ten *= 10;

            if (pr.second == true && check[temp]) answer++;
        }

    } while(next_permutation(arr, arr+len));

    return answer;
}
~~~
