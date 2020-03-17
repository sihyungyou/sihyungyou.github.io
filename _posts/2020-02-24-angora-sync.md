---
layout: post
title: "Distributed Angora Slave's Internal State Synchronization"
tags: [Fuzzing, Angora, 캡스톤]
comments: true
---

> 어렵다 어려워!!  

## Problem Definition  
Master fuzzer에서 Slave fuzzer들에게 일을 나누어 주는 것은 어려운 일이 아니다. 단순한 TCP통신으로 테스팅할 타겟 프로그램과 입력으로 주어질 시드 디렉토리정도만 넘겨주고 fuzzer를 실행시키는 정도이다. 하지만 여러 slave fuzzer들이 각자 뻐징을 하는 것은 결국 여러번 할 뻐징을 한번에 하는 것과 다름이 없다. 물론 24시간씩 3일을 돌릴 것을 세 서버에서 24시간만 돌리면 되므로 시간을 절약하는데는 도움이 된다. 하지만 이걸로는 "fuzzing performance 향상" 측면에서는 전혀 진전이 없다. 궁극적으로 slave들이 뻐징을 하는 과정에서 나오는 새로운 input, path 정보를 유기적으로 주고받으며 더 효율적으로 버그를 검출할 수 있도록 해야한다.  

## Single CPU에서는..  
한 컴퓨터에서 여러 스레드로 fuzzing을 하면 메인 스레드와 자식 스레드가 각자 별개로 돌아간다. 자식 스레드들은 각각 executor 객체를 생성하여 알맞는 fuzz type에 따라서 fuzzing을 진행하고 그 과정에서 생기는 새로운 input 파일을 queue 디렉토리에 저장한다. 그런데 sync를 맞춘다고 했다. 여러 스레드들이 동시에 한 디렉토리에 접근해서 각자가 찾은 새로운 입력파일들을 id:0000n의 형태로 저장하려다보면 파일명이 겹치지 않을까? 앙고라에서는 이를 id 변수를 AtomicSize type으로 선언해서 방지한다. 그래서 사실 여러 스레드들이 가진 파일끼리 비교해서 "싱크를 맞춘다"라기 보다는 애초에 저장할 때 충돌이 없도록 방지하는 것이다.  

## Distributed Environment 에서는 어떻게 해야할까..  
Angora는 새로운 input을 생성하면 queue 디렉토리에 id:00001과 같은 형태의 파일로 저장한다. 먼저 각 slave에서 새로운 정보를 찾아낼 때 마다 slave의 queue 디렉토리와 master의 queue 디렉토리의 sync를 맞춰주고 거기서 적절하게 다른 slave들에게 input을 배분할 수 있다. sync를 맞추는 건 rsync를 사용하면 매우 쉽다. 다만, 중복되는 정보와 정보의 배분이 문제다.  

예를 들어 slave1에서 id:00020을 찾아서 master에게 복사해줬다고 하자. 먼저 이 파일과 같은 파일이 master에 있는지 검사해야한다. 더하여, 이 시드를 다른 slave들에게 나누어주기 전에 확인할 것이 있는데 다른 slave가 이미 그 input을 가지고 있는지 여부다. 즉 겹치는 시드를 줘봤자 같은 내용의 파일을 다른 이름으로 뻐징하는 것 밖에 되지 않으므로 전혀 쓸모가 없는 것이다. rsync 커맨드는 파일내용이 같아도 (diff로 검사해서 차이가 없어도) 파일이름이 다르면 전송하기 때문에 이와같은 검사절차가 필요하다.  

(2020.03.05 추가)  
slave fuzzer들은 동시에 새로운 internal state를 각자 발견하고 fuzzing을 진행하기 때문에 master로 new status를 전송했을 때 그 정보들이 atomic 하게 관리, 저장되어야 한다. (위에서 언급한 중복된 파일에 대해서는 일단 생각하지 않기로 한다) 그리고 분배는 new status를 발견하면 master로부터 atomic한 id번호를 부여받은 후 모든 slave에게 전부 뿌려준다.  

(2020.03.17 추가)  
일단 우리 팀원들이 동의한 internal state 정의는 다음의 네 정보들이다 : queue, depot(hang, crash, input), stats, branches. 분산환경에서 이 네가지를 master, slave 모두 동일하게 갖고있어야 sync를 맞췄다고 할 수 있다. 그런데 여기서 의문점이 하나 있다. 동일한 input을 가지고 fuzzing을 하는 것이라면 performance 면에서 single fuzzer와 차이가 있을까? 같은 input을 가지고도 여러 random mutation을 통해서라면 다른 결과를 낼 가능성이 높은건가?  