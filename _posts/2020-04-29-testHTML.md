---
layout: post
title: "실험결과 정리"
tags: [Fuzzing, Angora, 캡스톤]
comments: true
---

> Original Angora 실험결과  

### Setup  
먼저 testAutomat는 seed가 매우 부족해서 (다섯개 뿐이다) 같은 libxml의 프로그램인 testHTML를 타겟으로 잡고 진행했다. 더하여 linux binutil 프로그램 중 하나인 objdump 프로그램도 실험했다. num jobs는 4로, fuzzing 시간은 4시간, timestamp logging interval은 10초 단위이며 총 10번 반복해서 실험했다. 

### Graph  
실험 결과 그래프는 아래와 같다. 다음으로 진행할 것은 다른 타겟프로그램으로 같은 실험하는 것과 distributed Angora가 더 나은 성능을 내는지 보는 것이다. 하나의 타겟 프로그램만으로 실험결과를 정리할 수는 없는데 path를 찾는 것은 프로그램 특성이 반영되기 때문이다. 여러 프로그램에 대해서 그래프를 그려보고 baseline을 잡아갈 예정이다.  

### testHTML  
![Center example image](https://user-images.githubusercontent.com/35067611/80626214-0ce97300-8a89-11ea-81ec-9964722d6615.png "Center"){: .center-image}  

### objdump  
![Center example image](https://user-images.githubusercontent.com/35067611/80993979-f8392080-8e76-11ea-972a-0464f7926a4a.png "Center"){: .center-image}  