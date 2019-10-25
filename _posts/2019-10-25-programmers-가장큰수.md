---
layout: post
title: "프로그래머스 고득점 kit : 가장 큰 수"
tags: [알고리즘, 프로그래머스, 정렬, 문자열처리]
comments: true
---

> Programmers  

### 문제설명  
0 또는 양의 정수가 주어졌을 때, 정수를 이어 붙여 만들 수 있는 가장 큰 수를 알아내 주세요.  

예를 들어, 주어진 정수가 [6, 10, 2]라면 [6102, 6210, 1062, 1026, 2610, 2106]를 만들 수 있고, 이중 가장 큰 수는 6210입니다.  

0 또는 양의 정수가 담긴 배열 numbers가 매개변수로 주어질 때, 순서를 재배치하여 만들 수 있는 가장 큰 수를 문자열로 바꾸어 return 하도록 solution 함수를 작성해주세요.  

제한 사항  
- numbers의 길이는 1 이상 100,000 이하입니다.  
- numbers의 원소는 0 이상 1,000 이하입니다.  
- 정답이 너무 클 수 있으니 문자열로 바꾸어 return 합니다.  

입출력 예  
~~~
numbers             return
[6, 10, 2]          6210
[3, 30, 34, 5, 9]   9534330
~~~

### 접근  


### 코드  
~~~c++
#include <string>
#include <vector>
#include <algorithm>
#include <iostream>
#include <utility>

using namespace std;

bool sortbysec (pair<string, string> &a, pair<string, string> &b) {
    return (a.second < b.second);
}

string solution(vector<int> numbers) {
    string answer = "";
    vector < pair <string, string> > v;
    string temp, temp2;
    int i, j, len;
    
    
    for (i = 0; i < numbers.size(); i++) {
        temp2 = "";
        
        temp = to_string(numbers[i]);
        temp2.assign(temp);
        
        if (temp2.length() == 1) {
            for (j = 0; j < 3; j++) temp2.push_back(temp2[0]);
        } else if (temp2.length() == 2) {
            temp2 += temp2;
        } else if (temp2.length() == 3) {
            temp2.push_back(temp2[0]);
        }
        v.push_back(make_pair(temp, temp2));
    }
    
    sort(v.begin(), v.end(), sortbysec);
    
    for (i = v.size()-1; i >= 0; i--) answer += v[i].first;
    
    return answer;
}
~~~