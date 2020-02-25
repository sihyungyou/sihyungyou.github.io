---
layout: post
title: "분산된 Slave 서버간의 Sync 맞추기"
tags: [Fuzzing, Angora, 캡스톤]
comments: true
---

> rsync  

## Problem Definition  
Master fuzzer에서 Slave fuzzer들에게 일을 나누어 주는 것은 어려운 일이 아니다. 단순한 TCP통신으로 테스팅할 타겟 프로그램과 입력으로 주어질 시드 디렉토리정도만 넘겨주고 fuzzer를 실행시키는 정도이다. 하지만 여러 slave fuzzer들이 각자 뻐징을 하는 것은 결국 여러번 할 뻐징을 한번에 하는 것과 다름이 없다. 물론 24시간씩 3일을 돌릴 것을 세 서버에서 24시간만 돌리면 되므로 시간을 절약하는데는 도움이 된다. 하지만 이걸로는 "fuzzing performance 향상" 측면에서는 전혀 진전이 없다. 궁극적으로 slave들이 뻐징을 하는 과정에서 나오는 새로운 input, path 정보를 유기적으로 주고받으며 더 효율적으로 버그를 검출할 수 있도록 해야한다.  

## 어떻게 해야할까  
Angora는 새로운 input을 생성하면 queue 디렉토리에 id:00001과 같은 형태의 파일로 저장한다. 먼저 각 slave에서 새로운 정보를 찾아낼 때 마다 slave의 queue 디렉토리와 master의 queue 디렉토리의 sync를 맞춰주고 거기서 적절하게 다른 slave들에게 input을 배분할 수 있다. sync를 맞추는 건 rsync를 사용하면 매우 쉽다. 다만, 중복되는 정보와 정보의 배분이 문제다.  

예를 들어 slave1에서 id:00020을 찾아서 master에게 복사해줬다고 하자. 먼저 이 파일과 같은 파일이 master에 있는지 검사해야한다. 더하여, 이 시드를 다른 slave들에게 나누어주기 전에 확인할 것이 있는데 다른 slave가 이미 그 input을 가지고 있는지 여부다. 즉 겹치는 시드를 줘봤자 같은 내용의 파일을 다른 이름으로 뻐징하는 것 밖에 되지 않으므로 전혀 쓸모가 없는 것이다. rsync 커맨드는 파일내용이 같아도 (diff로 검사해서 차이가 없어도) 파일이름이 다르면 전송하기 때문에 이와같은 검사절차가 필요하다.  