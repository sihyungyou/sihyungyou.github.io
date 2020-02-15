---
layout: post
title: "Angora Fuzzer : Executor의 역할 이해하기"
tags: [Fuzzing, Angora]
comments: true
---

> 앙고라 뻐저 파헤치기  

앙고라 코드를 분석하다보면 Executor가 심심치 않게 보인다. 하나의 구조체로 executor.rs에 정의되어있는데 실제 fuzzing을 실행하는 fuzz_loop() 함수에서도 새롭게 선언되고 여러 스레드들이 sync를 맞추는 main_thread_sync_and_log() 함수에도 전달되는 등 여기저기 빠지는 곳이 없다. 뭔가 중요한 친구 같아서 fuzz_main()부터 시작해서 쭉 따라가보았다.  

## 최초 선언 시점  
가장 먼저 선언되는 시점은 fuzz_main()이다. 이 때 주목해야 할 것은 인자로 command_option.specify(0)가 넘어가는데 이 함수가 멀티 스레딩단계에서 중요하다.

## 뭐하는 친구일까  
실제로 fuzzing을 돌리고 그 결과로 나오는 것들을 저장하고 관리하는 역할을 한다. 오늘은 그 중에서도 forksrv를 생성하는 부분에 집중해서 공부했다.

## 앙고라 멀티스레딩의 진실  
위에서 언급한 command_option.specify 함수에서는 out_dir path, forksrv name, 그리고 전달받은 id(위의 경우에는 0)를 concat 하여 소켓을 만든다. 예를 들면 `forksrv_socket_1`과 같은 이름을 가진 소켓이 생성된다. 이렇게 만들어진 소켓이 Unixlistener와 bind되고 새로운 client가 있다면 accept하기도 한다. 그리고 가장 중요한 것은 child process를 spawn해서 target program을 실행시키는 것이다.  

즉, 일단 fuzz_main에서 id 0으로 하나가 생성되고 angora_fuzzer를 실행시킬 때 option으로 주는 num jobs의 개수만큼 소켓이 추가로 더 생성된다. num_jobs == 4라면 (id 0인 메인스레드를 제외하고) 네 개의 스레드가 생성되고 각각이 free cpu core와 bind된다. 그리고 fuzz_loop() 단계로 넘어가면서 executor가 생성되고 이전과 같이 소켓을 만드는 과정이 반복된다. 이 소켓들도 모두 각각 child process를 spawn하여 target program execution이 이루어진다. 실제로 서로 다른 pid를 가진 프로세스들이 실행되는 모습을 확인할 수 있었다. 다음은 libxml Automata 프로그램을 네 개의 프로세스로 나누어 각자 다른 input으로 fuzzing하도록 한 것이다.  

![Center example image](https://user-images.githubusercontent.com/35067611/74587593-17729f80-5038-11ea-9629-2dd573940007.png "Center"){: width="400" height="400"}  

Concurrency란 싱글코어에서 멀티스레딩이 이루어지는 것을 의미하고 Parallelism은 멀티코어에서 각각 멀티스레딩이 진행되는 것이다. 앙고라는 num jobs만큼 스레드를 생성하고 free cpu core와 bind한다. 그 부분이 의아했다. 왜냐하면 멀티스레딩을 할거면 새로운 코어가 아니라 지금 쓰고있는 코어에서 돌려도 충분하기 때문이다. 그런데 오늘 child process가 여러개 생기면서 target binary에 대해 뻐징을 진행한다는 것을 알고나니 왜 스레드를 cpu core에 붙여주는지 조금이나마 이해가 됐다. CPU는 한 순간에 하나의 프로세스만 처리할 수 있으므로 여러 프로세스를 관리하기위해서는 코어가 더 필요했던 것 같다(?)  

## 더 생각해볼 점  
client가 없으면 socket이 accept를 하지 않는데 여기서 client가 들어온다는게 무슨 의미일까ㅏ  
