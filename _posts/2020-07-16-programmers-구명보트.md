---
layout: post
title: "프로그래머스 : 구명보트"
tags: [알고리즘, 프로그래머스, Greedy, Deque]
comments: true
---

> Programmers  

[구명보트](https://programmers.co.kr/learn/courses/30/lessons/42885)  

### 접근  
먼저 몸무게를 내림차순으로 정렬한다. 초기접근은 이렇게 정렬된 벡터를 O(N^2)로 탐색하는 알고리즘이었다. 가장 무거운 사람 한명을 선택하고 뒤에서부터 쭉 보트에 태울 수 있는 인원을 확인했다. 그리고 보트에 태운 사람은 -1로 표기하여 구분지었다. 하지만 벡터의 한계점은 erase를 하면 인덱스가 엉망이 되어서 구현이 쉽지 않아 O(N)으로 줄일 아이디어가 도무지 떠오르지가 않았다.  

질문하기에서 deque라는 힌트를 얻었다. 그렇다! deque를 이용하면 현 시점 가장 무거운 사람을 선택하고 뒤에서부터 보트에 태울 수 있는 인원들을 pop 해가면서 O(N)만에 문제를 해결할 수 있다. 정렬 후 각 시점에서 가장 무거운 사람을 선택하고 보트에 더 태울 수 있는 사람은 가벼운 순으로 선택한다는 관점에서 Greedy 문제였다.  

### O(N^2) 코드  
~~~c++
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

bool cmp(int a, int b) {
    return a > b;
}

int solution(vector<int> people, int limit) {
    int answer = 0;
    int i, j, k;
    int len = people.size();

    sort(people.begin(), people.end(), cmp);

    for (i = 0; i < len; i++) {
        if (people[i] == -1) continue;
        int temp = people[i];
        people[i] = -1;
        for (j = len - 1; j > i; j--) {
            if (temp + people[j] <= limit && people[j] != -1) {
                temp += people[j];
                people[j] = -1;
            }
        }
        answer++;
    }

    return answer;
}
~~~

### O(N) 코드  
~~~c++
#include <string>
#include <vector>
#include <algorithm>
#include <deque>

using namespace std;

bool cmp(int a, int b) {
    return a > b;
}

int solution(vector<int> people, int limit) {
    int answer = 0;
    int i, j, k;
    int len = people.size();
    deque <int> dq;

    sort(people.begin(), people.end(), cmp);
    for (i = 0; i < len; i++) dq.push_back(people[i]);

    while(!dq.empty()) {
        int front = dq.front();
        dq.pop_front();
        int temp = front;
        while(!dq.empty()) {
            int back = dq.back();
            if (temp + back <= limit) {
                temp += back;
                dq.pop_back();
            }
            else break;
        }
        answer++;
    }

    return answer;
}
~~~
