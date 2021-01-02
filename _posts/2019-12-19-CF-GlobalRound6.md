---
layout: post
title: "코드포스 Global Round 6 (2 Solved)"
tags: [알고리즘, 코드포스]
comments: true
---

> CodeForces  

### [A번 링크](https://codeforces.com/contest/1266/problem/A)  

### 접근  
배수 판정법을 이용한 수학문제다. 60의 배수는 3의배수일 조건 : 각 자리 숫자의 합이 3의 배수, 20의 배수일 조건 : 십의 자리가 0, 2, 4, 6, 8이고 일의 자리가 0인 수를 만족하면 된다.

### 코드  
~~~c++
#include <cstdio>
#include <cstring>

using namespace std;

int main() {

    int n, i, j, len, sum;
    int zero, even;
    bool three;
    scanf("%d", &n);

    for (i = 0; i < n; i++) {
        char in[101];
        zero = 0;
        even = 0;
        three = false;
        sum = 0;

        scanf("%s", in);
        len = strlen(in);
        for (j = 0; j < len; j++) {
            int temp = in[j] - '0';

            if (temp == 0) zero++;
            else if (temp % 2 == 0) even++;
            sum += temp;
        }
        if (sum % 3 == 0) three = true;
        if ( (zero >= 2 && three) || (zero >= 1 && three && even >= 1)) printf("red\n");
        else printf("cyan\n");
    }

    return 0;
}
~~~

### [B번 링크](https://codeforces.com/contest/1266/problem/B)  

### 접근  
주사위 하나를 놓으면 보이는 면의 합은 반드시 14의 배수 + 윗면(1~6) 이다. 그 이유는 주사위가 마주보는 면의 합이 7이고 사면이 보이기 때문이다. 그러니까 윗면을 제외하면 몇개의 주사위가 있든지 14의 배수여야한다.  

### 코드  
~~~c++
#include <cstdio>

using namespace std;

int main() {

    int n, i, j;
    long long num;
    scanf("%d", &n);

    for (i = 0; i < n; i++) {
        scanf("%lld", &num);

        for (j = 1; j < 7; j++) {
            if ( (num > 14) && ((num - j) % 14) == 0 ) {
                printf("YES\n");
                break;
            }
        }
        if (j == 7) printf("NO\n");
    }

    return 0;
}
~~~
