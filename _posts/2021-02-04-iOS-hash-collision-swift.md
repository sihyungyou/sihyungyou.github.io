---
layout: post
title: "iOS) Swift는 해시충돌을 어떻게 해결할까?"
tags: [iOS, Swift]
comments: true
---

> Cocoa Design Pattern - Singleton  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

면접에서 해시테이블 개념에 대해 질문을 받았다. 간단하게 key-value pair로 데이터를 저장하고 상수시간복잡도 내에 데이터를 찾을 수 있는 자료구조라고 답했는데, 문득 Swift에서는 해시충돌을 어떻게 해결할까? 라는 생각이 들었다. Swift에는 Set, Dictionary와 같이 hash value를 계산해서 메모리에 데이터를 저장하는 컬렉션을 제공하는데 아무리 좋은 해시함수라해도 무한한 입력에 대해 해시충돌을 완벽히 예방할 수는 없을 것이다.

Java를 예로 들면 jdk7까지는 linked list를 사용한 separte chaining 방법을, jdk8부터 linked list와 red black tree를 혼용한 separate chaining 방법을 활용한다고 하던데 (주워듣기로는) Swift는 어떻게 해시충돌을 해결하는지 궁금해졌다.

의외로 답은 금방 찾았다. Set에 대한 [미디엄 글](https://heartbeat.fritz.ai/diving-into-data-structures-in-swift-sets-e972c5a26b72)을 읽다가 "this is how Swift resolves collisions in sets and dictionaries—open addressing with linear probing" 이라는 문구를 발견했다. 또, [스택오버플로우](https://stackoverflow.com/questions/28379809/how-are-hash-collisions-handled)에서도 찾을 수 있었는데, 어떤 용자들이 `NSDictionary` 코드를 뜯어보고 결론 내리기를 역시 linear probing 방식으로 해시충돌을 해결한다고 한다.

"A this point, the only way to find out how Swift handles collisions would be disassembling the library (that is, unless you work for Apple, and have access to the source code). Curious people did that to NSDictionary, and determined that it uses **linear probing**, the simplest variation of the open addressing technique."

## Hashable, Equatable

여기서 한 스텝 더 나아가서 Swift에서 개발자가 직접 만드는 Custom Type을 Hashable하게 만드는 경우에 대해 생각해보자. 예를 들어 Person 이라는 클래스 객체를 만들고 Set으로 관리하거나 Dictionary의 key로 사용한다든지? 이렇게 개발자가 직접 정의하는 새로운 자료구조 중 서로 다른 값이지만 같은 key를 갖게 되었다고 하자. (hash function의 성능이 더 좋았으면 이런 일도 줄겠지만) insert 시점에는 위에서 알아본 바와 같이 linear probing을 통해 무리없이 데이터를 저장할 수 있다. 하지만 access 시점에는 결국 같은 key(hashValue)를 가진 객체간 비교를 통해 원하는 데이터를 찾아내야 할 것이다.

여기서 지금까지 "그냥" 해야되는 줄로만 알고 있던 `==` operation의 필요성, 즉 Hashable 프로토콜이 Equatable 프로토콜을 채택하는 깨달았다! (이제서야..) 해시 충돌이 발생하면 이제 hashValue가 아닌 다른 요소로 eaulity를 검사하는 것이다. 그래서 Hashable한 객체는 Equatable 프로토콜도 채택해야하고, 그러므로 `==` operation을 정의해야한다.

단, 이 메소드는 개발자가 자신이 정의한 객체의 성질을 잘 생각해서 정의해야한다. 예를 들어 Person 객체의 equality를 생각할 때 name, birthday가 같으면 같은 사람이라고 할 수 없다. 상식적으로 "같다"라고 판단하기에 더 많은 정보가 필요한 것이다. 이렇게 hashValue가 같을 때를 대비하여 객체 간의 "같음"에 대해 잘 정의하고 객체의 property를 이용해서 equality를 판단해야 한다.

## References

[Diving Into Data Structures In Swift: Sets](https://heartbeat.fritz.ai/diving-into-data-structures-in-swift-sets-e972c5a26b72)

[How are hash collisions handled?](https://stackoverflow.com/questions/28379809/how-are-hash-collisions-handled)

[How to handle hash collisions for Dictionaries in Swift](https://stackoverflow.com/questions/31664159/how-to-handle-hash-collisions-for-dictionaries-in-swift)
