---
layout: post
title: "프로그래머스 : 숫자의 표현"
tags: [알고리즘, 프로그래머스, 구현]
comments: true
---

> Programmers  

[숫자의 표현](https://programmers.co.kr/learn/courses/30/lessons/12924)  

### 접근  
구현을 위한 아이디어는 매우 쉽다. 더하기를 시작할 수를 1부터 n/2까지 증가시킨다. 각각의 덧셈이 시작되는 수부터 다시 반복문에 들어가 수를 누적시키며 더하고 n과 같아지는 시점에서 반복문을 탈출한다. 여기까지가 O(N^2) 시간복잡도이다. 아래의 코드를 보면 알 수 있겠지만 n/2까지만 더해도 되기 때문에 문제의 조건대로라면 내 코드는 최대 연산횟수가 2500만 정도이다. 그런데 TLE..  

누적시키면서 더해가는 과정에서 n보다 큰 경우에 바로 탈출하는 조건까지 추가했더니 효율성테스트까지 모두 통과했다. 그런데 O(2500만)이 TLE면 시간제한이 1초 미만인건가..? (그런거면 문제 제한사항에 써주지;) 보통 1억번 연산이 1초 내로 들어온다고 알고있는데 ㅠㅠ 무튼 DP로 풀어야하나 DFS로 풀어야 하나 고민을 많이했지만 결국 그냥 구현에 탈출조건 하나 추가하는 걸로 풀렸다. TLE 해결하는게 가장 어려운듯 흑흑  

### 코드  
~~~c++
using namespace std;

int solution(int n) {
    int answer = 0;
    int i, j, half = n/2;

    for (i = 1; i <= half; i++) {
        int temp = 0;
        for (j = i; j <= half + 1; j++) {
            temp += j;
            if (temp == n) {
                answer++;
                break;
            }
            // 여기서 break이 없다면 효율성 테스트 시간초과
            else if (temp > n) break;
        }
    }

    return answer + 1;
}

~~~
