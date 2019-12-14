---
layout: post
title: "191214 코드포스 B번 : Make Them Odd"
tags: [알고리즘, 대회, Greedy]
comments: true
---

> 코드포스  

### 문제설명  
time limit per test : 3 seconds  
memory limit per test : 256 megabytes  

There are 𝑛 positive integers 𝑎1,𝑎2,…,𝑎𝑛. For the one move you can choose any even value 𝑐 and divide by two all elements that equal 𝑐.  

For example, if 𝑎=[6,8,12,6,3,12] and you choose 𝑐=6, and 𝑎 is transformed into 𝑎=[3,8,12,3,3,12] after the move.  

You need to find the minimal number of moves for transforming 𝑎 to an array of only odd integers (each element shouldn't be divisible by 2).  

Input  
The first line of the input contains one integer 𝑡 (1≤𝑡≤104) — the number of test cases in the input. Then 𝑡 test cases follow.  

The first line of a test case contains 𝑛 (1≤𝑛≤2⋅105) — the number of integers in the sequence 𝑎. The second line contains positive integers 𝑎1,𝑎2,…,𝑎𝑛 (1≤𝑎𝑖≤109).  

The sum of 𝑛 for all test cases in the input doesn't exceed 2⋅105.  

Output  
For 𝑡 test cases print the answers in the order of test cases in the input. The answer for the test case is the minimal number of moves needed to make all numbers in the test case odd (i.e. not divisible by 2).  

### 접근  
수열이 [40, 6, 40, 20, 3, 1]로 주어졌다고 하자. 문제의 핵심은 40이 5로 나누어지는 과정에 20을 포함한다는 것과 이 검사를 TLE에 걸리지 않도록 하는 것이다. 먼저 벡터로 정렬을 하고 중복원소를 제거하면 TLE이므로 처음부터 입력을 받을 때 홀수는 거르고 set 자료구조를 이용하여 중복값 제거와 정렬을 알아서 하도록 해준다. 그리고 큰 값부터 홀수가 될 때 까지 기존값을 set 에서 제거하고 새로운 값(기존값의 1/2)을 넣는다. 이 때 set의 특성 상 이미 있는 값은 들어가지 않으므로 중복하여 횟수를 세는 것을 방지할 수 있다.  

### 코드  
~~~c++
#include <cstdio>
#include <set>
using namespace std;

int main() {

    int t, n, i, j, temp, a, cnt = 0;

    scanf("%d", &t);

    for (i = 0; i < t; i++) {
        set<int> s;
        cnt = 0;
        scanf("%d", &n);

        for (j = 0; j < n; j++) {
            scanf("%d", &temp);
            if (temp % 2 == 0) s.insert(temp);
        }

        set<int>::iterator k;
        while(!s.empty()) {
            k = s.end();
            k--;

            a = *k;
            while(a % 2 == 0) {
                s.erase(a);
                a /= 2;
                if (a % 2 == 0)s.insert(a);
                cnt++;
            }
        }

        printf("%d\n", cnt);
        
    }


    return 0;
}
~~~