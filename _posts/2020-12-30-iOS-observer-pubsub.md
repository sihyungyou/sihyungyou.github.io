---
layout: post
title: "iOS) Observer vs. Pub/Sub 패턴"
tags: [iOS]
comments: true
---

> Design Pattern  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

흔히 옵저버 패턴과 Pub/Sub 패턴을 동일하게 생각하는 경우가 많다. 실제로 작동하는 방식에 있어서 관찰자와 관찰 대상으로 역할이 나뉘고 데이터에 변경이 생길 경우 관찰자에게 notify하는 흐름은 매우 유사하다. 하지만 분명히 이 둘은 서로 다른 디자인 패턴으로 차이점이 명확히 존재한다. 오늘은 이 두 패턴에 대해 알아보겠다.

## 옵저버 패턴

The observer pattern is a software design pattern in which an object, called the subject, maintains a list of its dependents, called observers, and notifies them automatically of any state changes, usually by calling one of their methods.

옵저버 패턴은 subject(관찰 대상)이 observers(관찰자)를 list로 관리하며 상태 변화가 일어났을 때 옵저버의 메소드를 호출하면서 알리는(notify) 디자인 패턴이다.

## Pub/Sub 패턴

In ‘Publisher-Subscriber’ pattern, senders of messages, called publishers, do not program the messages to be sent `directly` to specific receivers, called subscribers.

반면 Pub/Sub 패턴은 notify 과정이 직접적이지(direct) 않다고 한다. 이게 무슨 뜻일까? Pub/Sub 패턴에서는 관찰대상과 관찰자 외에 브로커(broker) 혹은 버스(event bus)라고 불리는 세번째 컴포넌트가 존재한다. 이 컴포넌트는 publisher, subscriber 모두에게 알려져있으며 둘 사이의 커뮤니케이션을 담당하는 역할을 한다. 즉 옵저버 패턴과의 차이점은 관찰대상과 관찰자 사이에 둘을 연결하는 또 다른 컴포넌트가 있으며 이로 인해 `관찰대상과 관찰자가 서로에 대해 전혀 몰라도 무방`하다는 것이다.

## 비교하기

위에서 설명한 두 패턴의 차이점을 명확히 나타내는 그림을 보자.

![1](https://user-images.githubusercontent.com/35067611/104592252-3e56ad00-56b1-11eb-9170-3c99c3e64b35.png)

유사한 디자인 패턴이지만 중간에 이벤트 채널이 하나 들어감으로써 많은 차이점을 만들어낸다. 하나씩 살펴보자.

### 1. Observer와 Subject가 서로를 인지하는가?

옵저버 패턴에서는 필수적으로 그렇다. 하지만 Pub/Sub 패턴에서는 중간에 다리 역할을 하는 이벤트채널이 있기 때문에 Publisher와 Subscriber가 서로 인지하지 못해도(직접적인 참조관계를 갖지 않아도) 충분히 상태 변화를 감지할 수 있다.

### 2. 결합도 차이

1번 내용과 자연스럽게 연결되는 내용이다. Pub/Sub 패턴에서는 관찰대상과 관찰자가 서로의 존재를 알 필요가 없으므로 소스코드도 겹칠 일이 없다. 그러므로 당연히 결합도 측면에서 Observer 패턴보다 낮다고 할 수 있다. 이러한 디커플링은 더 다이나믹한 네트워크 토폴로지와 높은 확장성을 허용한다.

### 3. 단일 도메인 vs. 크로스 도메인

역시 중간에 매개체가 존재함으로써 얻을 수 있는 이점이다. Pub/Sub 패턴의 경우 도메인이 다르더라도 중간의 Publisher와 Subscriber가 각각 이벤트 채널에만 접근할 수 있다면 처리가 가능하다.

### 4. 동기 vs. 비동기

옵저버 패턴의 경우 대부분 동기 방식으로 이벤트 발생 시, 객체가 모든 옵저버의 적절한 메서드들을 호출한다. 반면 Pub/Sub 패턴은 비동기 메시징 패러다임이다. 중간의 이벤트 채널이 발행된 정보를 Subscriber에게 나눠주므로 이벤트를 발행만 해놓고 독립적인 다른 작업을 수행해도(비동기적이어도) 문제가 없다.

## References

[Observer vs Pub-Sub pattern - Hacker Noon](https://hackernoon.com/observer-vs-pub-sub-pattern-50d3b27f838c)
