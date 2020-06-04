---
layout: post
title: "프로그래머스 고득점 kit : 단어변환"
tags: [알고리즘, 프로그래머스, DFS]
comments: true
---

> Programmers  

### 문제설명  
두 개의 단어 begin, target과 단어의 집합 words가 있습니다. 아래와 같은 규칙을 이용하여 begin에서 target으로 변환하는 가장 짧은 변환 과정을 찾으려고 합니다.  
~~~
1. 한 번에 한 개의 알파벳만 바꿀 수 있습니다.
2. words에 있는 단어로만 변환할 수 있습니다.
~~~
예를 들어 begin이 hit, target가 cog, words가 [hot,dot,dog,lot,log,cog]라면 hit -> hot -> dot -> dog -> cog와 같이 4단계를 거쳐 변환할 수 있습니다.  

두 개의 단어 begin, target과 단어의 집합 words가 매개변수로 주어질 때, 최소 몇 단계의 과정을 거쳐 begin을 target으로 변환할 수 있는지 return 하도록 solution 함수를 작성해주세요.  

제한사항  
각 단어는 알파벳 소문자로만 이루어져 있습니다.  
각 단어의 길이는 3 이상 10 이하이며 모든 단어의 길이는 같습니다.  
words에는 3개 이상 50개 이하의 단어가 있으며 중복되는 단어는 없습니다.  
begin과 target은 같지 않습니다.  
변환할 수 없는 경우에는 0를 return 합니다.  

### 접근  
모든 경우의 수를 재귀함수를 통해 전부 검사하는 DFS 문제다 (완탐과 DFS는 묘하게 귀에걸면 귀걸이 코에걸면 코걸이인 느낌이다..) 가장 먼저 해야할 일은 target이 주어진 벡터에 있는지 확인하는 것이다. 없다면 DFS를 시작할 필요도 없이 변환하지 못하는 경우로 처리하면 되기 때문이다. DFS의 종료조건은 타겟을 찾았을 때와 words 벡터를 모두 탐색했을 때다. 재귀적으로 다음 DFS를 부르는 조건은 한 개의 알파벳만 다른 경우이다.  

한 가지 신경써야하는 부분은 재귀를 종료할 때 visit 배열에서 현재 검사하고 있는 단어에 대해 방문했다고 체크했던 것을 다시 방문하지 않음으로 돌려놓아야 한다는 것이다. 그래야 다른 경우의 수에서 words 벡터를 탐색하며 변환 가능한지에 대해 올바르게 체크할 수 있다.

### 코드  
~~~c++
#include <string>
#include <vector>
#include <algorithm>
#include <climits>

using namespace std;

vector<string> w;
string t;
int answer = INT_MAX;
int vec_len;
int word_len;
bool visit[51] = { false, };

void dfs(int idx, int n) {

    int i, j;
    string s = w[idx];
    visit[idx] = true;

    if (s.compare(t) == 0) {
        // found target
        answer = min(answer, n);
        visit[idx] = false;
        return;
    }

    for (i = 0; i < vec_len; i++) {
        string word = w[i];
        int difference = 0;

        for (j = 0; j < word_len; j++) {
            if (s[j] != word[j]) {
                difference++;
            }
        }

        if (difference == 1 && !visit[i]) {
            dfs(i, n + 1);
        }
    }

    visit[idx] = false;
    return;
}

int solution(string begin, string target, vector<string> words) {
    int i;
    t = target;

    word_len = target.length();

    w.push_back(begin);
    for (i = 0; i < words.size(); i++) {
        w.push_back(words[i]);
    }

    vec_len = w.size();

    if (find(words.begin(), words.end(), target) == words.end()) {
        return 0;
    }
    else {
        dfs(0, 0);
        return answer;
    }
}
~~~
