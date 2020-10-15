---
layout: post
title: "프로그래머스 : 디스크컨트롤러"
tags: [알고리즘, 프로그래머스, Heap]
comments: true
---

> Programmers  

[디스크 컨트롤러 문제링크](https://programmers.co.kr/learn/courses/30/lessons/42627)  

### 접근  


### 코드  
~~~c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <queue>
using namespace std;

bool cmp(vector<int> a, vector<int> b) {
    // 요청시간기준 오름차순 정렬
    return a[0] < b[0];
}

struct cmpStruct
{
    // min heap
    bool operator()(vector<int> a, vector<int> b) {
        return a[1] > b[1];
    }
};

int solution(vector<vector<int>> jobs) {
    int n = jobs.size();
    int answer = 0;

    // 1. 현재시간은 0이다
    int t = 0;

    // 2. 전체를 요청시간기준으로 정렬한다
    sort(jobs.begin(), jobs.end(), cmp);

    // 3. min heap을 선언한다
    priority_queue <vector<int>, vector<vector<int> >, cmpStruct> pq;

    while(jobs.size() > 0 || !pq.empty()) {
        while(jobs.size() > 0) {
            vector<int> currentJob = jobs[0];
            // 5. 현재시간 t가 처음에 들어있는 작업보다 크거나 같다면 해당 작업을 min heap에 넣고, 벡터에서는 삭제한다
            if (t >= currentJob[0]) {
                pq.push(currentJob);
                jobs.erase(jobs.begin());
            } else {
                break;
            }
        }

        if (!pq.empty()) {
            // 6. min heap에 작업이 있다면 작업 하나를 처리한다 -> t가 바뀐다
            vector<int> doingJob = pq.top();
            pq.pop();
            t += doingJob[1];
            answer += t - doingJob[0];
        } else {
            // pq가 비었다면 바로 다음 작업을 시작할 수 있도록 t를 다음작업의 요청시간으로 업데이트시킨다
            // 이 코드가 없다면 시간초과
            t = jobs[0][0];
        }

        // 7. 변경된 t를 갖고 4번으로 돌아가 반복한다
    }

    answer /= n;

    return answer;
}
~~~
