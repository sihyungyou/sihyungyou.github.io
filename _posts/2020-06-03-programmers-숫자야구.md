---
layout: post
title: "프로그래머스 고득점 kit : 숫자야구"
tags: [알고리즘, 프로그래머스, Brute-force]
comments: true
---

> Programmers  

### 문제설명  
숫자 야구 게임이란 2명이 서로가 생각한 숫자를 맞추는 게임입니다.  

각자 서로 다른 1~9까지 3자리 임의의 숫자를 정한 뒤 서로에게 3자리의 숫자를 불러서 결과를 확인합니다. 그리고 그 결과를 토대로 상대가 정한 숫자를 예상한 뒤 맞힙니다.  
~~~
* 숫자는 맞지만, 위치가 틀렸을 때는 볼
* 숫자와 위치가 모두 맞을 때는 스트라이크
* 숫자와 위치가 모두 틀렸을 때는 아웃
~~~
예를 들어, 아래의 경우가 있으면

~~~
A : 123
B : 1스트라이크 1볼.
A : 356
B : 1스트라이크 0볼.
A : 327
B : 2스트라이크 0볼.
A : 489
B : 0스트라이크 1볼.
~~~

이때 가능한 답은 324와 328 두 가지입니다.  

질문한 세 자리의 수, 스트라이크의 수, 볼의 수를 담은 2차원 배열 baseball이 매개변수로 주어질 때, 가능한 답의 개수를 return 하도록 solution 함수를 작성해주세요.  

제한사항  
질문의 수는 1 이상 100 이하의 자연수입니다.  
baseball의 각 행은 [세 자리의 수, 스트라이크의 수, 볼의 수] 를 담고 있습니다.  

### 접근  
진혁이가 "형 할 수 있어요"라고 했지만 생각이 도저히 안나서 아니 조건 처리 어떻게 하는거냐고 라는 마인드로 방법에 대해서 구글링을 해보았다. 결국 찐완탐이었다. 숫자야구에서 나올 수 있는 수는 각 자리의 수가 모두 다르고 0이 포함되지 않는 세자리 수다. 그러므로 123~987까지의 모든 경우에 대해 문제에서 주어진 조건에 부합하는지를 하나하나 맞춰보는 문제였다. 조건에 맞는지 체크하는 방법은 자리수도 같고 숫자도 같으면 strike 변수를, 자리수는 다르지만 숫자가 같다면 ball 변수를 하나씩 올리고 주어진 배열의 스트라이크와 볼과 일치하는지 검사한다. 만약 baseball 배열의 모든 조건들을 만족한다면 answer 변수를 증가시킨다.  

### 코드  
~~~c++
#include <string>
#include <vector>

using namespace std;

int solution(vector<vector<int>> baseball) {
    int answer = 0;
    int blen = baseball.size();
    int i, j;

    for (i = 123; i <= 987; i++) {
        int one = i % 10;
        int ten = (i / 10) % 10;
        int hun = (i / 100) % 10;

        if (one != 0 && ten != 0 && hun != 0 && one != ten && one != hun && ten != hun) {

            for (j = 0; j < blen; j++)  {
              int strike = 0;
              int ball = 0;
              int b = baseball[j][0];
              int bone = b % 10;
              int bten = (b / 10) % 10;
              int bhun = (b / 100) % 10;

              if (one == bone) strike++;
              if (ten == bten) strike++;
              if (hun == bhun) strike++;
              if (one == bhun || one == bten) ball++;
              if (ten == bhun || ten == bone) ball++;
              if (hun == bone || hun == bten) ball++;
              if (strike != baseball[j][1] || ball != baseball[j][2]) break;
            }

            if (j == blen) answer++;
        }
    }

    return answer;
}
~~~
