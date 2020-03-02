---
layout: post
title: "Angora Fuzzer 소개"
tags: [실프2]
comments: true
---

> 앙고라 소개하기  

## Fuzzer  
앙고라에 대해 소개하기 전에 먼저 `fuzzing`이 무엇인지부터 말해야한다. 앙고라가 fuzzing tool, fuzzer의 한 종류이기 때문이다. fuzzing의 정의는 다음과 같다.  

> invalid, unexpected, random data를 입력으로 받아 버그 테스팅을 하는 기술  

즉 `software testing`을 하는 기술이다. 사람이 만든 많은 소프트웨어는 문제가 없어 보이지만 완전무결할 수 없다. 더 문제인 것은 눈에 보이지 않는 버그가 존재할 수 있다는 가능성을 알면서도 그 버그를 trigger할 입력을 만들어내는 것이 매우 어렵다는 것이다. 그래서 위에서 언급했듯 invalid, unexpected, random 데이터를 넣는 것이다. 그러나 여기서 random이 완전 무작위로 입력을 넣는다는 것을 의미하지는 않는다. 그럴 경우 의미있는 테스팅을 하기도 전에 프로그램이 에러처리를 해버리는 경우가 있기 때문이다.  

예를 들어 url 링크를 입력으로 받아 parsing하는 프로그램이 있다고 해보자. 이 프로그램은 http, https로 시작하는 입력만 valid하다고 간주하고 그 외의 경우는 모두 에러 핸들링으로 처리해버린다. 이런 프로그램에 무작위 입력을 넣는다면 버그를 찾기는커녕 아예 프로그램 진행조차 해보지 못하고 에러로 빠질 것이다. 즉, invalid란 해당 프로그램이 run 할 수 있도록 요구되는 최소한의 조건과 형태를 갖춘 입력상태에서 약간의 수정을 통해 버그를 찾아내기위한 invalidity를 의미한다.  

이렇게 fuzzing을 하는 tool은 이미 많이 개발되어있는데 가장 널리 쓰이는 AFL(American Fuzzy Lop), Vuzzer, Angora 등 오픈소스로 공개되어 여러 분야에서 활용되고 있다.  

## Angora Fuzzer  
앙고라는 여러 fuzzer중 하나로 mutation-based coverage guided fuzzer이다. 가장 널리 쓰이는 AFL과의 차이점은 네 측면에서 볼 수 있다. Context-sensitive branch coverage, Scalable byte-level taint tracking, Search based on gradient descent, Type and Shape interface 이다. 이 네 기술 모두 fuzzing performance를 높이기 위한 노력이다. 더 빨리, 더 많은 버그를 찾아내기 위한 것이다.  