---
layout: post
title: "iOS) 타 언어의 GC와 Swift ARC의 차이"
tags: [iOS, Swift]
comments: true
---

> 그놈의 ARC  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

Swift에서는 Automatic Reference Counting 이라는 메모리 관리 모델을 사용한다. Swift와 마찬가지로 다른 언어들 (예를 들어 Java) 역시 각자의 메모리 관리 모델이 있는데 대표적인 것이 GC(garbage collection)이다. ARC와 GC는 어떻게 다르고 애플은 왜 ARC를 선택했을까? (기존에 맥에서 지원되던 가비지컬렉션조차 없애버렸다고 한다, 그리고 가비지 컬렉션을 사용한 앱은 심사에서 거부해버린다) 이 둘의 차이점에 대해 알아보자

## GC

먼저 `GC` 방식은 메모리 관리를 Garbage Collector(가비지 컬렉터)라는 것이 프로그램 실행중에 동적으로 감시하고 있다가, 더 이상 사용할 필요가 없다고 여겨지는 것을 메모리에서 삭제하는 것이다. 여기서 가장 중요한 포인트는 `실행타임`에서 메모리 관리를 한다는 것이다.

## ARC

이와 달리, `ARC`는 프로그램이 실행되고 있는 상태에서 감시하는 것이 아니라 코드를 빌드(`컴파일`)할 때 컴파일러가 프로그래머 대신에 release 코드를 적절한 위치에 넣어주는 개념이다. 그러니까 Swift 코드에서 개발자가 release 시키는 코드를 작성하지 않아도 실제로 바이너리 코드에는 이런 메모리 해제 코드가 들어가도록 한다는 것이다.

## GC vs. ARC

이것은 아주 중요한 차이점이다. 단순히 런타임이나 컴파일타임이라는 시점의 차이가 만들어내는 또 다른 차이가 있기 때문이다. 가비지 컬렉션은 항상 메모리를 차지하고 감시해야하기 때문에 프로그램 자체 외에 메모리 사용량이 더 늘어날 수 밖에 없으며 지속적인 감시를 위해 CPU를 일부 사용해야만한다. 이에 비해, ARC는 수동으로 개발자가 넣을 코드를 컴파일러가 넣어주는 것이기에 그런 오버헤드에서 자유롭다는 것이다. 이는 특히 메모리와 CPU가 데스크탑에 비해 제한적인 모바일 기기에서는 (당연히) 더 중요한 문제이고 그만큼 성능 측면에서 이득이다.  

## 정리

GC

- 참조 계산 시점 : 런타임. 어플 실행 중 주기적으로 참조를 추적하여 사용하지 않는 인스턴스를 해제한다.
- 장점 : 인스턴스가 해제될 확률이 RC에 비해 높다.
- 단점 : 개발자가 참조 해제 시점을 파악할 수 업음, 런타임 시점에 계속 추적하는 오버헤드로 성능저하가 있을 수 있다.

RC

- 참조 계산 시점 : 컴파일타임. 컴파일 시점에 언제 참조되고 해제되는지 "결정"되어 런타임때 "실행"된다.
- 장점 : 개발자가 참조 해제 시점을 파악할 수 있다. 런타임 시점에 추가적인 오버헤드가 발생하지 않는다.
- 단점 : 순환참조 발생 시 영구적으로 메모리가 해제되지 않아 메모리 누수의 위험이 있다.

## GC는 순환참조를 해결할 수 있을까?

면접에서 받은 질문이다. (예방할 수 있냐고 물어보신 것 같기도 한데 정확히 기억이 안난다) 일단 ARC 방식으로는 retain cycle이 발생하면 해결할 수 없다. 메모리 누수가 발생하는 것이다. 그렇다면 GC는 어떨까? 일단 GC가 작동하는 방식에 대해 생각해보자. 위에서는 "더 이상 사용할 필요가 없다고 여겨지는 것을 메모리에서 삭제하는 것" 정도로 퉁치고 말았는데 이 질문에 대해서 답하기 위해서는 내부 메커니즘에 대한 이해도 필요하다.

GC는 Mark and Sweep 프로세스를 통해 메모리에서 필요없는 부분을 해제한다. 아래 그림을 보자.
![mark-and-sweep](https://user-images.githubusercontent.com/35067611/106987013-ed1d7300-67af-11eb-8836-70c559828235.png)

가비지 컬렉터에는 GC root라는 것이 있는데 힙 외부에서 접근할 수 있는 변수나 오브젝트를 뜻하며 말그대로 가비지 컬렉션의 root 역할을 한다. 이 루트에서 시작하여 참조로 연결된 오브젝트들을 모두 마크(mark) 한다. 그리고 마크가 끝나면 힙 내부 전체를 돌면서 마크되지 않은 객체들을 모두 해제(reclaim)한다. 이 과정을 sweep이라고 하기 때문에 mark and sweep이라고 부르는 것이다. 간단히 정리하면 루트로부터 닿을 수 있는(rechable) 노드들을 살려놓고 나머지 객체는 메모리에서 해제하는 것이다.

그럼 이 메커니즘을 순환참조와 연결지어 생각해보자. 결론부터 말하자면 GC 방식은 순환참조가 발생해도 메모리 누수를 방지할 수 있다. 일반적으로 순환참조가 메모리 누수라는 결과로 이어지는 이유는 객체끼리 서로 참조하는데 각 객체로 접근하는 외부 참조가 사라져 이 순환참조를 제거할 수 없기 때문이다. 이 때 외부 참조가 사라졌다는 것은 GC 루트로부터 닿을 수 없다는(unrechable) 것이다. 위에서 언급했듯, mark and sweep 과정은 루트로부터 참조를 타고 닿을 수 있는 객체를 제외하고는 그 객체들이 순환참조를 하든, 얼마나 많은 노드를 포함하는 그래프의 형태를 띄고 있든 메모리에서 해제한다고 했다. 그러므로 GC는 순환참조가 발생해도 메모리 누수로 이어지지 않게 되는 것이다.

## References

[ARC는 가비지컬렉션이 아니예요.](https://wingsnote.com/32)

[소들이님 블로그 - iOS) 메모리 관리 (1/3) - ARC(Automatic Reference Counting)](https://babbab2.tistory.com/26)

[가비지 컬렉터(Garbage Collector)와 Mark & Sweep](https://imasoftwareengineer.tistory.com/103)

[StackOverflow - How does Java solve retain cycles in garbage collection?](https://stackoverflow.com/a/20993931)
