---
layout: post
title: "Codeforces Global Round #6 A번 : Competitive Programmer"
tags: [알고리즘, 대회, 코드포스, 수학]
comments: true
---

> 코드포스  

### 문제설명  
Bob is a competitive programmer. He wants to become red, and for that he needs a strict training regime. He went to the annual meeting of grandmasters and asked 𝑛 of them how much effort they needed to reach red.  

"Oh, I just spent 𝑥𝑖 hours solving problems", said the 𝑖-th of them.  

Bob wants to train his math skills, so for each answer he wrote down the number of minutes (60⋅𝑥𝑖), thanked the grandmasters and went home. Bob could write numbers with leading zeroes — for example, if some grandmaster answered that he had spent 2 hours, Bob could write 000120 instead of 120.  

Alice wanted to tease Bob and so she took the numbers Bob wrote down, and for each of them she did one of the following independently:  

rearranged its digits, or wrote a random number. This way, Alice generated 𝑛 numbers, denoted 𝑦1, ..., 𝑦𝑛.  

For each of the numbers, help Bob determine whether 𝑦𝑖 can be a permutation of a number divisible by 60 (possibly with leading zeroes).  

Input  
The first line contains a single integer 𝑛 (1≤𝑛≤418) — the number of grandmasters Bob asked.  

Then 𝑛 lines follow, the 𝑖-th of which contains a single integer 𝑦𝑖 — the number that Alice wrote down.  

Each of these numbers has between 2 and 100 digits '0' through '9'. They can contain leading zeroes.  

Output  
Output 𝑛 lines.  

For each 𝑖, output the following. If it is possible to rearrange the digits of 𝑦𝑖 such that the resulting number is divisible by 60, output "red" (quotes for clarity). Otherwise, output "cyan".  

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