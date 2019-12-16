---
layout: post
title: "Codeforces Round #606 A번 : Happy Birthday, Polycarp!"
tags: [알고리즘, 대회, 코드포스, 구현]
comments: true
---

> 코드포스  

time limit per test : 1 second  
memory limit per test : 256 megabytes  

Hooray! Polycarp turned 𝑛 years old! The Technocup Team sincerely congratulates Polycarp!  

Polycarp celebrated all of his 𝑛 birthdays: from the 1-th to the 𝑛-th. At the moment, he is wondering: how many times he turned beautiful number of years?  

According to Polycarp, a positive integer is beautiful if it consists of only one digit repeated one or more times. For example, the following numbers are beautiful: 1, 77, 777, 44 and 999999. The following numbers are not beautiful: 12, 11110, 6969 and 987654321.  

Of course, Polycarpus uses the decimal numeral system (i.e. radix is 10).  

Help Polycarpus to find the number of numbers from 1 to 𝑛 (inclusive) that are beautiful.  

Input  
The first line contains an integer 𝑡 (1≤𝑡≤104) — the number of test cases in the input. Then 𝑡 test cases follow.  

Each test case consists of one line, which contains a positive integer 𝑛 (1≤𝑛≤109) — how many years Polycarp has turned.  

Output  
Print 𝑡 integers — the answers to the given test cases in the order they are written in the test. Each answer is an integer: the number of beautiful years between 1 and 𝑛, inclusive.  

### 접근  
구현문제인데 디테일을 잡지 못해서 애를 먹었다.. 반성하자 ㅜㅜ  

beautiful number의 조건은 모든 자리수가 같아야 한다. 세자리수 180이 온다면 1~9, 11~99는 자동으로 포함하고 111~999 중에 몇 개를 포함하는지 알아보면 된다. 이와 같이 몇 개를 포함하는지 알아보기 위해서는 모든 자리수에 대해 첫번째 자리의 수 이상인지 검사한다. 예를들어 180에서 18까지는 1 이상이므로 111을 포함할 가능성이 있다. 마지막 0은 1 미만이다. 이 경우엔 두번째 자리부터 1 미만의 수 직전까지 모두 검사하며 1을 초과하는 수가 적어도 하나 있으면 111을 포함할 수 있다. 180은 8이 1을 초과하므로 마지막 자리수가 1미만이지만 111을 포함하여 한 개를 더해준다. 하지만 110의 경우, 두번째 1이 첫번째 자리수를 초과하지 못하여 111을 포함하지 않아 18개이다.  

### 코드  
~~~c++
#include <cstdio>
#include <cstring>

using namespace std;

int main() {

    int t, i, j, k, len, first, cnt = 0;
    bool flag;

    scanf("%d", &t);

    for (i = 0; i < t; i++) {
        char in[11];
        scanf("%s", in);
        len = strlen(in);
        cnt = 0;
        first = in[0] - '0';
        flag = true;


        if (len == 1) {
            cnt += first;
        }
        else {
            cnt += (len-1) * 9;

            for (j = 1; j < len; j++) {
                if (in[j] < in[0]) {
                    for (k = 1; k < j; k++) {
                        if (in[k] > in[0]) break;
                    }
                    if (k == j) {
                        flag = false;
                        break;
                    }
                }
            }

            if (flag) {
                cnt += first;
            } else {
                cnt += first - 1;
            }
        }

        printf("%d\n", cnt);
    }



    return 0;
}
~~~