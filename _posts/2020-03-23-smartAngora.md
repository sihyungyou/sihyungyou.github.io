---
layout: post
title: "Angora의 Smart Parallelization 위한 idea들"
tags: [일상]
comments: true
---

> 캡스톤 아자아자  

일단 지금까지의 진행상황은 이렇다. Angora는 처음 오픈소스를 다운로드 받아 사용할 때 부터 이미 multi-core fuzzing은 가능했다. 하지만 이것을 distributed environment로 확장하고, multi-core, multi-process 레벨까지 구현했다. fuzzing을 실제로 진행하는 fuzz_loop 함수에서는 executor를 생성하고 이를 SearchHandler에 담아서 이것저것(?)을 하는데 이 지점에서 Slave CPU들이 각각 갖고있는 local stat, branch 정보를 master의 global stat, branch로 넘겨오는 것 까지는 됐다.  

하지만 이는 단순한 브랜치 데이터, 혹은 스탯의 숫자들을 가져와서 add 해주는 것에 불과하다. 
