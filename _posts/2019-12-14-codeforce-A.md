---
layout: post
title: "191214 코드포스 A번 : Happy Birthday, Polycarp!"
tags: [알고리즘, 대회, Greedy]
comments: true
---

> BOJ  

/*
A. Happy Birthday, Polycarp!
time limit per test1 second
memory limit per test256 megabytes
inputstandard input
outputstandard output
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

Example
input
6
18
1
9
100500
33
1000000000

output
10
1
9
45
12
81
*/

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