---
layout: post
title: "프로그래머스 : 전화번호 목록"
tags: [알고리즘, 프로그래머스, Hash]
comments: true
---

> Programmers  

[전화번호 목록](https://programmers.co.kr/learn/courses/30/lessons/42577)  

### 접근  
가장 쉽게 떠올릴 수 있는 로직은 2중 반복을 통한 substring 여부 검사다. 아래의 첫번째 코드인데 이렇게 코드를 짤 경우 문제에서 주어지는 phone_book의 길이가 1이상 1000000이하이므로 TLE에 걸리게 된다. 그래서 map(해시)를 사용해서 시간복잡도를 줄여야한다.  

모든 전화번호들의 모든 substring을 map에 넣는다. 예를 들어 12345라는 전화번호가 있다면 ```1```, ```12```, ```123```, ```1234```을 해시테이블에 넣는 것이다. 해시테이블을 만드는 작업이 모두 끝나면 다시 처음부터 모든 전화번호들을 탐색하며 해당 전화번호가 map에 있는지 (즉 다른 전화번호의 substring인지) 여부를 판단한다.  

### O(N^2) 코드  
~~~c++
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

bool solution(vector<string> phone_book) {
    bool answer = true;
    int i = 0;
    int j = 0;
    int len = phone_book.size();
    sort(phone_book.begin(), phone_book.end());

    for (i = 0; i < len - 1; i++) {
        for (j = i + 1; j < len; j++) {
            if (phone_book[j].find(phone_book[i]) != string::npos) {
                answer = false;
                break;
            }
        }
    }
    return answer;
}
~~~

### O(N) 코드  
~~~c++
#include <string>
#include <vector>
#include <map>

using namespace std;

bool solution(vector<string> phone_book) {
    bool answer = true;
    int i, j, len = phone_book.size();
    map <string, int> m;

    // map에 모든 전화번호들의 모든 substr을 넣는다 -> O(N * 20)
    for (i = 0; i < len; i++) {
        int length = phone_book[i].length();
        for (j = 1; j < length; j++) {
            string temp = phone_book[i].substr(0, j);
            m[temp] = 1;
        }
    }

    // map에서 전화번호를 O(1)로 찾고 있다면 다른 전화번호의 substr이라는 뜻이므로 false, break -> O(N)
    for (i = 0; i < len; i++) {
        map <string, int>::iterator it = m.find(phone_book[i]);
        if (it != m.end()) {
            answer = false;
            break;
        }
    }

    return answer;
}
~~~
