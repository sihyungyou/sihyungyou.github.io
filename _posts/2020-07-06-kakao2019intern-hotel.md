---
layout: post
title: "프로그래머스 : 호텔 방 배정"
tags: [알고리즘, 프로그래머스, Disjoint-set]
comments: true
---

> 2019 카카오 개발자 겨울 인턴십 문제  

[호텔 방 배정](https://programmers.co.kr/learn/courses/30/lessons/64063#)  

### 접근  
초기접근은 배열을 사용한 union-find 알고리즘의 사용이었다. 하지만 배열을 사용하면 메모리 부족으로 효율성 테스트를 통과할 수 없다. k가 10^12인데 배열을 k개만큼 선언해야 하기 때문이다. 배열 대신 map을 사용하면 k개가 아니라 room_number의 크기만큼의 (이 문제에서는 최대 200000) 방 정보만 관리하여 문제를 풀 수 있다. 즉 포인트는 호텔이 보유한 방의 개수는 k개인데 실제로 투숙객은 room_number.size() 밖에 안된다는 것이다.  

map을 떠올렸다면 나머지 구현은 union-find 알고리즘을 안다면 어렵지 않다. 배정한 방 번호를 key, 배정한 방 보다 크면서 비어있는 방 중 가장 번호가 작은 방을 value로 find와 union 함수를 구현하면 된다. union은 다음 방 번호를 find한 후 map에 insert만 하면 되고, find는 path compression까지 생각해줘야 시간복잡도를 통과할 수 있다.  

### 코드  
~~~c++
#include <vector>
#include <map>

using namespace std;

map <long long, long long> m;

long long do_find(long long x) {
    if (m.find(x) == m.end()) {
        // 만약 x가 map의 key로 존재하지 않는다면 -> return x
        return x;
    }
    // 존재한다면 -> 그 key의 value를 인자로 do_find를 다시 진행
    // path compression
    return m[x] = do_find(m[x]);
}

void do_union(long long x, long long y) {
    // map에 추가
    x = do_find(x);
    y = do_find(y);
    m[x] = y;
}

vector<long long> solution(long long k, vector<long long> room_number) {
    long long want, checkin;
    vector<long long> answer;
    int i, len = room_number.size();

    for (i = 0; i < len; i++) {
        want = room_number[i];
        checkin = do_find(want);
        answer.push_back(checkin);
        do_union(checkin, checkin + 1);
    }

    return answer;
}
~~~
