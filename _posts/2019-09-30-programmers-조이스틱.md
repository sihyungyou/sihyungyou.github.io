---
layout: post
title: "프로그래머스 고득점 kit : 조이스틱"
tags: [알고리즘, 프로그래머스, Greedy]
comments: true
---

> Programmers  

### 문제설명  
조이스틱으로 알파벳 이름을 완성하세요. 맨 처음엔 A로만 이루어져 있습니다.  
ex) 완성해야 하는 이름이 세 글자면 AAA, 네 글자면 AAAA  

조이스틱을 각 방향으로 움직이면 아래와 같습니다.  
~~~
▲ - 다음 알파벳  
▼ - 이전 알파벳 (A에서 아래쪽으로 이동하면 Z로)  
◀ - 커서를 왼쪽으로 이동 (첫 번째 위치에서 왼쪽으로 이동하면 마지막 문자에 커서)  
▶ - 커서를 오른쪽으로 이동  
~~~

예를 들어 아래의 방법으로 JAZ를 만들 수 있습니다.
~~~
- 첫 번째 위치에서 조이스틱을 위로 9번 조작하여 J를 완성합니다.  
- 조이스틱을 왼쪽으로 1번 조작하여 커서를 마지막 문자 위치로 이동시킵니다.  
- 마지막 위치에서 조이스틱을 아래로 1번 조작하여 Z를 완성합니다.  
~~~
따라서 11번 이동시켜 "JAZ"를 만들 수 있고, 이때가 최소 이동입니다.  

만들고자 하는 이름 name이 매개변수로 주어질 때, 이름에 대해 조이스틱 조작 횟수의 최솟값을 return 하도록 solution 함수를 만드세요.  

제한 사항  
- name은 알파벳 대문자로만 이루어져 있습니다.  
- name의 길이는 1 이상 20 이하입니다.  

### 접근  
각 위치의 알파벳을 만들기 위해서 몇 번을 움직여야하는지 알아내는 방법은 어렵지 않다. 예를 들어 J라면 J-A는 ASCII 값으로 9이다. 이건 A부터 오른쪽으로 움직였을때 경우인데, 거꾸로 오는 경우는 26-(J-A)번이다. 즉 min함수만 적용하면 된다.  

하지만 문자열에 A 혹은 여러 연속된 A가 있다면 조이스틱을 왼쪽, 오른쪽으로 움직여야한다. 이 부분은 아직 생각중...

### 코드  
~~~c++
#include <string>
#include <vector>

using namespace std;

int min(int a, int b) { return a < b ? a : b; }

int solution(string name) {
    int answer = 0;
    int i = 0;
    int len = name.length();
    int arr[len];
    
    for (i = 0; i < len; i++) {
        int temp = name[i] - 'A';
        arr[i] = min(temp, 26-temp);
        answer += arr[i];
    }
    
    
    return answer;
}
~~~