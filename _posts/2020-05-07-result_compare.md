---
layout: post
title: "실험결과 비교"
tags: [Fuzzing, Angora, 캡스톤]
comments: true
---

> Angora, P-Angora, D-Angora 성능비교  

### P-Angora, D-Angora 
먼저 우리가 증명하고자 했던 명제를 다시 보면 다음과 같다. "distributed angora는 fuzzing의 성능을 효과적으로 향상시키는 방향으로 추가적으로 주어진 computing resoruce를 활용하는 알고리즘을 갖고있다." 여기서 말하는 distributed angora가 D-Angora이다. 즉, master 1대와 slave 4대가 동시에 fuzzing을 진행하면서 주기적으로 conditional statment를 담고 있는 priority queue, global branches, stats, generated population의 sync를 맞춘다.  

P-Angora는 D-Angora의 전 단계, 즉 parallelization만 이루어진 상태이다. 여기서는 초기 seed만 동일하고 slave의 fuzzing 결과를 master에게 update 할 뿐이지 다른 internal state를 동기화하지는 않는다. 즉 n대의 machine에서 Angora fuzzer가 각자 돈다고 생각하면 된다.  

### 결과비교  
objdump 프로그램을 타겟으로 Angora, P-Angora, D-Angora의 성능을 비교한 그래프이다.  

![Center example image](https://user-images.githubusercontent.com/35067611/81269008-aca79200-9083-11ea-8a57-4aba9abfec4d.png "Center"){: .center-image}  

![Center example image](https://user-images.githubusercontent.com/35067611/81269086-cea11480-9083-11ea-91db-dd0d99911b7b.png "Center"){: .center-image}  

5000초대까지 D-Angora가 치고 올라가는 것은 매우 좋았으나 그 이후에 갑자기 path를 찾는 양이 줄어든다. 그리고 결국엔 P-Angora보다 path 측면에서 역전당한다. 이런 현상의 원인으로는 Angora 내부의 FuzzType에 따른 fuzzing 진행방식때문이라고 분석된다.  

fuzzing을 하기 위해 queue에서 가장 높은 priority의 conditional statement를 가져와서 어떤 fuzz type을 갖고 있는지 확인한다. Explore, Exploit, Len, AFL 등의 fuzz type이 다양하게 있는데 이 중 random mutation 방식인 AFL이 선택되면 taint analysis의 이점을 활용하지 못해서 new path를 찾는 빈도가 상대적으로 적을 것이다. Angora가 진행될 때 stat을 보면 처음에는 Explore, Exploit fuzztype의 execution time이 상당히 높은데 갈수록 AFL type이 많아진다. 그래서 D-Angora의 그래프가 실험 초기에는 가파르게 올라가지만 갈수록 path를 찾지 못하는 것이다. 그런데 왜 P-Angora는 꾸준히 올라갈까?  

이 두번째 질문에 대한 원인은 population 공유여부라고 생각된다. AFL의 random mutation 방식은 n대의 machine들이 각각 자신의 local seed로 fuzzing 할 때 효과가 좋을 것이다. P-Angora가 이런 behavior를 갖고 있다. 반면, D-Angora는 newly generated input을 주기적으로 master와 모든 slave가 공유하도록 구현되어있다. 이럴 경우, 중복된 seed는 제거하고 AFL의 random mutation 방식으로 fuzzing할 seed 개수가 상대적으로 적어서 실험 후반으로 갈수록 성능이 좀처럼 올라가지 않는 것으로 생각된다.