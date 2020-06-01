---
layout: post
title: "프로그래머스 고득점 kit : 네트워크"
tags: [알고리즘, 프로그래머스, DFS]
comments: true
---

> Programmers  

### 문제설명  
네트워크란 컴퓨터 상호 간에 정보를 교환할 수 있도록 연결된 형태를 의미합니다. 예를 들어, 컴퓨터 A와 컴퓨터 B가 직접적으로 연결되어있고, 컴퓨터 B와 컴퓨터 C가 직접적으로 연결되어 있을 때 컴퓨터 A와 컴퓨터 C도 간접적으로 연결되어 정보를 교환할 수 있습니다. 따라서 컴퓨터 A, B, C는 모두 같은 네트워크 상에 있다고 할 수 있습니다.  

컴퓨터의 개수 n, 연결에 대한 정보가 담긴 2차원 배열 computers가 매개변수로 주어질 때, 네트워크의 개수를 return 하도록 solution 함수를 작성하시오.  

### 입출력 예시  
~~~
n	computers	                        return
3	[[1, 1, 0], [1, 1, 0], [0, 0, 1]]	2
3	[[1, 1, 0], [1, 1, 1], [0, 1, 1]]	1
~~~

### 접근  
단순한 DFS 문제다. 아직 방문하지 않은 노드에 대해서 DFS를 돌고 그때마다 카운트를 올려주면 된다.  

### 코드  
~~~c++
#include <string>
#include <vector>

using namespace std;

bool visit[200] = { false, };
vector<vector<int>> map;
int len;

void dfs(int v) {
    visit[v] = true;

    int i;
    for (i = 0; i < len; i++) {
        if (map[v][i] == 1 && !visit[i]) dfs(i);
    }
    return;
}

int solution(int n, vector<vector<int>> computers) {
    int answer = 0;
    len = n;

    for (int i = 0; i < n; i++) {
        map.push_back(computers[i]);
    }

    for (int i = 0; i < n; i++) {
        if (!visit[i]) {
            dfs(i);
            answer++;
        }
    }

    return answer;
}
~~~