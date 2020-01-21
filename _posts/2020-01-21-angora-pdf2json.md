---
layout: post
title: "[오픈소스테스팅 with Angora] pdf2json"
tags: [Fuzzing]
comments: true
---

> 앙고라로 뻐징하기  

## 문제점과 해결방법  
[pdf2json](https://github.com/flexpaper/pdf2json)은 이름에서 알 수 있듯 pdf 파일을 json 형태로 바꾸는 conversion program이다. pdf도 하나의 binary 형태를 가진 파일이므로 입력으로 주었을 때 변환 과정에서 crash가 나기를 기대할 수 있다. 그런데 AFL로 fuzzing 할 때와 다르게 Angora는 "Seed discarded, too long" 이라는 메세지를 출력하면서 fuzzing 진행을 하지 못했다. input pdf들은 커봤자 1MB 정도였다.  

그래서 아주 작은 크기의 pdf 파일을 입력으로 주었을 때는 none constraint가 없다며 fuzzing을 중단해버렸다. 결국 "적절한" 크기의 pdf를 찾아 성공했다. Angora 내부에 MAX INPUT LENGTH가 15000으로 되어있는데 약 13KB 입력에 대해 성공한 것으로 보아 15KB미만의 입력에 대해서만 정상적인 fuzzing을 하는 듯 하다. 그렇다면 AFL과 달리 이렇게 입력 크기에 민감하게 제한을 둔 이유가 무엇일까?  

## Angora 내부이해  
AFL이 하지 않고 Angora는 하는 것이 있다. Byte-level taint analysis이다. 입력의 어떤 byte offset이 unexplored branch b로 가는 데 영향을 주는지 정보를 표시해놓는 과정이다. 예를 들어 z = x + y 라고 한다면 affect(z) 즉, z에 영향을 미치는 요소들은 x, y, affect(x), affect(y)를 모두 포함하는 set 이다. 이렇게 거슬러 올라가다보면 그 끝에는 constant 혹은 input read일 것이다. 이렇게 추적하여 영향을 미치는 부분만 mutate 하겠다는 전략이다. unexplored branch로 가는데 영향을 주지 않는 byte offset에 대해 mutation 하지 않으므로 훨씬 효율적일것만 같으나 이 taint tracking 과정 자체가 매우 expensive 하다.  

공간적으로 expensive한 이유를 생각해보자. taint를 했으면 그 정보를 어딘가에 담아두어야 하는데 Angora는 그 taint label을 bit vector의 형태로 저장한다. bit vector의 i번째 bit는 입력의 i번째 byte offset을 의미한다. 그러므로 k byte input이 들어오면 k 만큼의 bit vector를 가지게되며 이런 Angora의 구조는 input length에 민감할 수 밖에 없는 환경인 것이다. 위에서 언급한 pdf 파일이 크기때문에 reject 당한 것도 이런 이유때문이라고 짐작된다.

## run Angora  
command  
~~~
./angora_fuzzer -i ~/cockatiel/sampe_pdfs/ -o pdf2json -t ~/pdf2json_taint/src/pdf2json -- ~/pdf2json_fast/src/pdf2json @@
~~~

![Center example image](https://user-images.githubusercontent.com/35067611/72793236-8e0cbf00-3c7d-11ea-8659-fcf808c3c647.png "Center"){: .center-image}  
12시간 fuzzing 한 결과 19개의 crash 발견  

## debugging  
![Center example image](https://user-images.githubusercontent.com/35067611/72794017-a204f080-3c7e-11ea-8a42-ded778b0c992.png "Center"){: .center-image}  
bt 결과가 만 개 이상으로 계속 내려가는 현상이 발생한다.. 이것도 좀 알아봐야할 듯