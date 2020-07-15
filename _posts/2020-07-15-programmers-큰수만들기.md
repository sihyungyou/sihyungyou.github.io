---
layout: post
title: "프로그래머스 : 큰 수 만들기"
tags: [알고리즘, 프로그래머스, Greedy]
comments: true
---

> Programmers  

[큰 수 만들기](https://programmers.co.kr/learn/courses/30/lessons/42883)  

### 접근  
이 풀이는 내 힘으로 풀지 못하고 [hyeon_Hwang님의 블로그](https://medium.com/hyeon-hwang/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%A8%B8%EC%8A%A4-%ED%81%B0-%EC%88%98-%EB%A7%8C%EB%93%A4%EA%B8%B0-lv-1-42883-%EC%88%AB%EC%9E%90-%EB%AC%B8%EC%A0%9C-%EA%B7%B8%EB%A6%AC%EB%94%94-585ce3b8c604)를 참고했다.  

블로그에 들어가보면 써있겠지만 내 말로 풀어 설명을 하자면 문자열의 뒤에서부터 ```반환해야할 문자열의 길이 - 1```만큼 제외하고 가장 큰 수를 찾는다. 이것은 곧 정답의 가장 앞에 올 수를 찾는 과정이다. 그리고 찾아 놓은 가장 큰 수의 ```다음 위치```부터 ```반환해야할 문자열의 길이 - 2```만큼 제외하고 다시 가장 큰 수를 찾는다. 정답의 두번째 자리에 올 수를 찾는 과정이다. 이것을 일반화하면 다음과 같다.  
~~~
[index, length - return_length] 범위의 가장 큰 수를 찾아 정답 문자열에 append한다.
index = 가장 큰 수의 위치 + 1로 업데이트, return_length를 1 감소시킨 후 위 과정을 반복한다.
~~~

### 코드  
~~~c++
#include <string>
#include <vector>

using namespace std;

string solution(string number, int k) {
    string answer = "";
    int len = number.length();
    int return_len = len - k;
    int i = 0;
    int idx = 0;
    char num;

    while(return_len > 0) {
        num = '0';
        for (i = idx; i < len - return_len + 1; i++) {
            if (number[i] > num) {
                idx = i+1;
                num = number[i];
            }
        }
        return_len--;
        answer += num;
    }

    return answer;
}
~~~
