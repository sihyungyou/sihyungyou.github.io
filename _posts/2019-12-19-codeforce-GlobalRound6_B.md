---
layout: post
title: "Codeforces Global Round #6 B번 : Dice Tower"
tags: [알고리즘, 대회, 코드포스, 수학]
comments: true
---

> 코드포스  

### 문제설명  
Bob is playing with 6-sided dice.  

He has an unlimited supply of these dice and wants to build a tower by stacking multiple dice on top of each other, while choosing the orientation of each dice. Then he counts the number of visible pips on the faces of the dice.

For example, the number of visible pips on the tower below is 29 — the number visible on the top is 1, from the south 5 and 3, from the west 4 and 2, from the north 2 and 4 and from the east 3 and 5.

![Center example image](https://user-images.githubusercontent.com/35067611/71168842-f04b5c00-229a-11ea-8259-8724041c6654.png "Center"){: .center-image}  

The one at the bottom and the two sixes by which the dice are touching are not visible, so they are not counted towards total.

Bob also has 𝑡 favourite integers 𝑥𝑖, and for every such integer his goal is to build such a tower that the number of visible pips is exactly 𝑥𝑖. For each of Bob's favourite integers determine whether it is possible to build a tower that has exactly that many visible pips.

Input
The first line contains a single integer 𝑡 (1≤𝑡≤1000) — the number of favourite integers of Bob.

The second line contains 𝑡 space-separated integers 𝑥𝑖 (1≤𝑥𝑖≤1018) — Bob's favourite integers.

Output
For each of Bob's favourite integers, output "YES" if it is possible to build the tower, or "NO" otherwise (quotes for clarity).

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