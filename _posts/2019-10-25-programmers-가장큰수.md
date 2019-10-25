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
참고로 이 문제는 테스트케이스 하나를 실패해서 90.9점을 받았다. 반례가 뭔지 더 생각해봐야겠다.. 알고리즘은 정렬자체보다는 문자열처리를 해주는 쪽에 가까운 것 같다.  

단순히 입력 숫자들을 정렬해서 이어붙이면 안되는데 그 이유는 위의 예시에서 3과 30의 쌍에서 330이 303보다 큰 수를 만들기 때문이다. 이런 예외경우를 처리하려면 언제 한자릿수가 그보다 큰 자릿수의 수보다 우선순위가 높은지 알아야한다. 이 문제에서 입력받는 원소는 최대 4자릿수이다. 그렇다면 3은 3333미만의 수보다는 먼저와야한다. 예를 들어, 3과 3331이 있다면 이 둘로 만들 수 있는 수는 33331과 33313이 있고 전자가 더 큰 수다.  

두자릿수의 경우는 끝에 첫자리와 두번째 자리를 append한 수보다 먼저올 수 있다. 예를 들어 12는 1212미만의 수보다 먼저올 수 있다. 12와 1211을 생각해보면 [12][1211], [1211][12]를 만들 수 있고 전자가 더 큰수다. 같은 논리로 세자릿수도 첫자리를 끝에 append한 수까지 커버할 수 있다.  

위 논리를 if문으로 처리해서 모든 수를 네자릿수로 만들고 정렬한 다음 순서에 맞게 원래의 수를 모두 합하면 된다.  

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