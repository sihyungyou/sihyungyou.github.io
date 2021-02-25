---
layout: post
title: "[번역] Method Dispatch in Swift"
tags: [iOS, Swift]
comments: true
---

> 메소드 디스패치에 대해 알아보자  

메소드 디스패치에 관련한 [글](https://medium.com/flawless-app-stories/static-vs-dynamic-dispatch-in-swift-a-decisive-choice-cece1e872d)을 번역한 글이다.

---

당신이 OOP를 자주 사용한다면 메소드 디스패치 기술이 꼭 새로운 것은 아닐 것이다. (특히 동적 디스패치는 더욱)

메소드 디스패치란 어떤 연산(operation)을 실행해야하는지 결정하도록 돕는 메커니즘으로, 더 정확히 말하자면 어떤 메소드 구현이 사용되어야하는지 결정하는 것이다.

## Basics

정적 디스패치는 값, 참조타입 모두에서 지원된다. 하지만 동적 디스패치는 참조타입에서만 지원된다. 이러한 이유는 동적 디스패치를 위해서는 상속이 필요한데 값 타입은 상속을 지원하지 않기 때문이다. 이 사실을 염두에 두고 다음 내용으로 가보자!

먼저 큰 그림을 그려보자면 사실 정적/동적 두 가지 타입보다 더 세분화해서 디스패치의 종류를 나눌 수 있다.

1. Inline
2. Static
3. Virtual
4. Dynamic

이 중 어떤 테크닉을 사용할 지는 컴파일러에 달려있다. 가장 빠른 Inline으로 올라가거나 가장 느린 Dynamic으로 내려오는 등 필요에 따라 결정한다.

## Static vs. Dynamic or Swift vs. Objective-C

디폴트로 Objective-C는 동적 디스패치를 지원한다. 이 디스패치 기술은 다형성의 형태로 프로그래머에게 상당한 유연성을 제공한다. 이미 존재하는 메소드 등을 서브 클래싱, 재정의할 수 있기 때문이다. 물론 매우 유용하지만 비용이 드는 과정이다.

동적 디스패치는 런타임 오버헤드를 들여 언어 표현력(expressivity)을 향상시킨다. 무슨 의미인가하면, 동적 디스패치에서의 모든 메소드 호출은 컴파일러가 witness table(혹은 virtual table)을 들여다보고 특정 메소드의 구현을 체크한 후에 이루어진다는 것이다. 컴파일러는 메소드 호출이 슈퍼클래스의 구현을 가리키는지, 서브 클래스의 구현을 가리키는지(refer to) 판단해야하기 때문이다. 그리고 메모리에 모든 오브젝트는 런타임에 올라가게 되므로 컴파일러는 이러한 과정을 런타임에 해야할 수 밖에 없다.

반면 정적 디스패치는 이러한 문제가 없다. 컴파일러는 컴파일 타임에 어떤 메소드 구현이 호출되어야할지 알고있다. 그러므로 컴파일러는 여러 최적화 기법을 적용할 수 있고 혹은 코드를 Inline으로 변환(convert)할 수도 있다. 따라서 전체적인 프로그램의 속도를 상당히 빠르게 향상시킬 수 있다.

### how do we achieve both of them in Swift?

동적 디스패치를 위해서 우리는 상속을 사용한다. 기본 클래스(base class)를 서브 클래싱하고 기본 클래스에 존재하는 메소드를 재정의(override)한다. 또한, 우리는 `dynamic` 키워드를 사용하고 `@objc` 키워드를 prefix하여 Objective-C 런타임에 메소드를 노출시키기도한다.

정적 디스패치를 위해서 우리는 `final`, `static` 키워드를 통해 클래스의 경우 추가적인 상속이나 재정의가 불가능하도록 만든다.

### Let's dive deep

### Static Dispatch (or Direct Dispatch)

위에서 설명되었듯 정적 디스패치는 컴파일러가 명령(instructions)들이 어디에 있는지 안다는 측면에서 (able to locate) 동적 디스패치보다 더 빠르다. 그러므로 함수가 호출되면 컴파일러는 수행할 함수의 메모리 주소로 바로(directly) jump 한다. 이는 성능을 크게 향상시키고 컴파일러가 inlining과 같은 최적화를 가능케한다.

### Dynamic Dispatch

동적디스패치는 컴파일 타임이 아닌 런타임에 어떤 메소드의 구현을 고를지 결정하기 때문에 정적 디스패치에 비해 오버헤드가 더 든다. 그런데 이렇게 비싼 작업이라면 애초에 사용하지 않으면 되는 것이 아닌가? 라는 생각이 들 수 있다.

이는 유연성 때문에 그렇다. 사실 대부분의 OOP 언어들은 동적 디스패치를 지원하는데 그 이유가 다형성을 위해서이다.

### 1. Table Dispatch

이 디스패치 기법은 테이블을 사용한다. 여기서 테이블이란 함수 포인터로 이루어진 배열을 의미하며 witness table (혹은 virtual table)이라고 불린다. 특정 메소드의 구현을 찾는(look up) 용도로 사용된다.

그렇다면 이 witness table이 어떻게 작동하는가?

- 모든 서브클래스는 각자의 테이블을 갖고있다.
- 이 테이블은 서브클래스가 재정의한 모든 메소드에 대해 다른 함수포인터를 갖고있다.
- 서브클래스가 새로운 메소드를 추가하면 해당 메소드에 대한 함수포인터가 배열 끝에 추가된다.
- 컴파일러가 런타임에 이 테이블을 사용하여 어떤 메소드를 호출할지 결정한다.

정적 디스패치와 달리 컴파일러가 먼저 테이블로부터 메모리 주소를 읽고(read) 해당 위치로 이동(jump)해야 하므로 두번의 연산이 요구된다. 이것이 정적 디스패치보다 느린 이유다. (메시지 디스패치보다는 여전히 빠르다)

### 2. Message Dispatch

이 동적 디스패치 기법은 가장 동적이라고 할 수 있다. 사실 이 기법은 매우 좋아서 (최적화 측면을 제외하면) Cocoa 프레임워크에서도 KVO, 코어데이터와 같은 곳에서 자주 사용된다.

또한 이 방법은 런타임에 메소드의 기능(functionality)를 바꾸는 `method swizzling`을 가능하게 한다. (Method Swizzling은 원래의 메소드를 runtime 때 원하는 메소드로 바꾸어 사용할 수 있도록 하는 기법)

Swift 컴파일러는 이 기법을 Obejctive-C 런타임을 사용해 achieve한다. 명시적으로 메시지 디스패치를 사용하기 위해서는 dynamic 키워드를 사용한다. Swift 4.0 이전에는 dynamic, `@objc` 키워드가 암시적(implicitly)으로 추가되었지만 그 이후부터는 `@objc` 키워드를 마크해주어야한다.

이런 메시지 디스패치를 위해 Objective-C 런타임을 사용하기 때문에 메시지가 디스패치 될 때 런타임은 클래스 위계(hierarchy)를 모두 보고(crawl) 실행할 메소드를 결정한다. 이 과정이 매우 느리기 때문에 성능향상을 위해 캐시를 제공하기도 한다.

### Examples

**값 타입**

```swift
struct Person {
    func isIrritating() -> Bool { } // Static
}

extension Person {
    func canBeEasilyPissedOff() -> Bool { } // Static
}
```

구조체와 열거형은 값타입이며 상속이 불가능하므로 컴파일러는 당연히 정적 디스패치를 사용할 것이다.

**프로토콜**

```swift
protocol Animal {
    func isCute() -> Bool { } // Table
}

extension Animal {
    func canGetAngry() -> Bool { } // Static
}
```

여기서 키포인트는 extension 내부에서는 항상 정적 디스패치가 이루어진다는 것이다. extension에서는 재정의가 불가능하기 때문에 그렇다.

**클래스 (참조타입)**

```swift
class Dog: Animal {
    func isCute() -> Bool { } // Table
    @objc dynamic func hoursSleep() -> Int { } // Message
}

extension Dog {
    func canBite() -> Bool { } // Static
    @objc func goWild() { } // Message
}

final class Employee {
    func canCode() -> Bool { } // Static
}
```

- 보통의 메소드 선언은 프로토콜과 동일하게 동적 디스패치(그 중에서도 테이블 디스패치)를 따른다.
- Objective-C 런타임에 노출시키면 (@objc 키워드를 통해) 메소드는 메시지 디스패치를 사용한다.
- 하지만 `final` 키워드로 마크되어 서브클래싱이 불가능하다면 클래스라도 정적 디스패치가 가능하다.

### Summary

![1](https://user-images.githubusercontent.com/35067611/108793871-e8d2c180-75c7-11eb-82a6-22ceb0d6f770.png)

## References

[Static vs Dynamic Dispatch in Swift: A decisive choice](https://medium.com/flawless-app-stories/static-vs-dynamic-dispatch-in-swift-a-decisive-choice-cece1e872d)
