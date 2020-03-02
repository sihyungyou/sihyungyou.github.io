---
layout: post
title: "Angora Fuzzer 사용기"
tags: [실프2]
comments: true
---

> 앙고라 활용 및 경험에 대한 후기  

## 설치  
설치가 무지막지하게 까다롭진 않으나 버전 등에 신경써야할 부분이 있다. 예를 들어 내가 사용하는 서버는 Ubuntu 18이어서 16버전에서 돌아가는 LLVM 보다 높은 버전의 것으로 재설치했어야 했다. 그리고 bashrc에서 경로설정 해줄 때 조금 주의해주면 된다.  

## 실제사용  
앙고라는 실행시키는 커맨드가 꽤나 긴 편이다. 사실 앙고라 뿐만 아니라 fuzzer를 실행시키기 위해서는 fuzzing 결과를 저장할 경로, 처음에 seed로 줄 input 파일 경로, 타겟 프로그램 등 요구되는 사항이 많아서 좀 복잡하다. 실제로 매우 다양한 OPTION, FLAGS를 줄 수 있어서 실행하기 전에 자세히 살펴볼 필요가 있다.  
![Center example image](https://user-images.githubusercontent.com/35067611/75655986-5a727b00-5ca6-11ea-9be2-64727e9a71f7.png "Center"){: .center-image}  

앙고라를 실행하는 커맨드의 한 예시는 다음과 같다.  
~~~bash
$ ./angora_fuzzer -i ~/libxml2-2.9.2_fast/test/automata -o output -t ~/libxml2-2.9.2_taint/testAutomata -- ~/libxml2-2.9.2_fast/testAutomata @@
~~~
위의 커맨드는 libxml의 automata 프로그램을 fuzzing하는 것이다. AFL과 다르게 Angora는 하나의 타겟 프로그램을 taint, fast 두 버전으로 나누어 컴파일 시킨 후에 fuzzing을 진행한다. taint는 taing tracking이 적용된 채 컴파일 된 것이다.  