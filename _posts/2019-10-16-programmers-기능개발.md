---
layout: post
title: "프로그래머스 고득점 kit : 기능개발"
tags: [알고리즘, 프로그래머스, Queue]
comments: true
---

> Programmers  

### 문제설명  
프로그래머스 팀에서는 기능 개선 작업을 수행 중입니다. 각 기능은 진도가 100%일 때 서비스에 반영할 수 있습니다.  

또, 각 기능의 개발속도는 모두 다르기 때문에 뒤에 있는 기능이 앞에 있는 기능보다 먼저 개발될 수 있고, 이때 뒤에 있는 기능은 앞에 있는 기능이 배포될 때 함께 배포됩니다.  

먼저 배포되어야 하는 순서대로 작업의 진도가 적힌 정수 배열 progresses와 각 작업의 개발 속도가 적힌 정수 배열 speeds가 주어질 때 각 배포마다 몇 개의 기능이 배포되는지를 return 하도록 solution 함수를 완성하세요.  

- 작업의 개수(progresses, speeds배열의 길이)는 100개 이하입니다.  
- 작업 진도는 100 미만의 자연수입니다.  
- 작업 속도는 100 이하의 자연수입니다.  
- 배포는 하루에 한 번만 할 수 있으며, 하루의 끝에 이루어진다고 가정합니다. 예를 들어 진도율이 95%인 작업의 개발 속도가 하루에 4%라면 배포는 2일 뒤에 이루어집니다.  

입력  
~~~
progresses	speeds  
[93,30,55]	[1,30,5]  
~~~

출력  
~~~
answer
[2,1]
~~~

### 접근  
배포가 하루에 한번만 되는데 앞에있는 기능보다 먼저 개발되었더라도 먼저 배포될 수 없기 때문에 매일 배포될 수 있는 upper bound가 있다. 그 upper bound는 자료구조 큐를 사용하면 쉽게 구현 가능하다. 먼저 큐의 front를 bound로 정한다. 그리고 그 bound를 초과하지만 않는다면 큐에서 기다리고 있는 모든 기능들은 그 bound와 같은 날 배포될 수 있다. 그러면 bound 이하의 기능들을 모두 큐에서 pop하면 자동적으로 bound를 초과하는 첫번째 기능이 큐의 가장 앞으로 옮겨질 것이다. 그리고 위의 과정을 반복하면 된다!  

### 코드  
~~~c++
#include <queue>
#include <vector>

using namespace std;

vector<int> solution(vector<int> progresses, vector<int> speeds) {
    vector<int> answer; 
    queue <int> q;
    
    int bound, rest, day, i, cnt = 0;
    
    for (i = 0; i < progresses.size(); i++) {
        rest = 100 - progresses[i];
        if (rest % speeds[i] == 0) day = rest/speeds[i];
        else day = rest/speeds[i] + 1;
        q.push(day);
    }
    
    bound = q.front();
    
    while(!q.empty()) {
        if (q.front() <= bound) {
            cnt++;
            q.pop();
        }
        else {
            bound = q.front();
            answer.push_back(cnt);
            cnt = 0;
        }
    }
    answer.push_back(cnt);
    return answer;
}
~~~