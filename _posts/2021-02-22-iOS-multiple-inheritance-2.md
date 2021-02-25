---
layout: post
title: "[번역] 다중상속 (2) - Apply to Swift"
tags: [iOS, Swift]
comments: true
---

> Swift에서의 다중상속  

[지난 포스팅](https://sihyungyou.github.io/iOS-multiple-inheritance-1/)에서 다중상속은 무엇이며 문제점과 (다중상속을 지원하는 언어 C++에서의) 해결방법을 알아보았다. 오늘은 언어 레벨에서 다중상속을 지원하지 않는 Swift에서는 이와 같은 기능을 어떻게 흉내내는지 알아보자. 이와 관련해서 잘 설명한 [글](https://www.vadimbulavin.com/multiple-inheritance-swift/)을 번역하며 공부해보자!

---

Swift에서는 다중상속을 지원하지 않지만 이를 가능케하는 여러 API를 제공한다. Swift에서 어떻게 다중상속을 구현하는지 살펴보자.

### Understanding Multiple Inheritance

다중상속은 클래스가 한 개 이상의 부모 클래스로부터 행동(behvavior), 속성(attribute)를 상속받을 수 있는 객체지향 컨셉이다. 단일상속, 합성과 함께 다중상속은 여러 클래스간 코드를 공유할 수 있도록 해준다.

### Multiple Inheritance in Swift

C++와 같은 몇몇 프로그래밍 언어에서는 다중상속이 기본적으로 제공되는 기능이지만 Swift는 그렇지 않다. Swift에서는 여러 프로토콜을 채택할 수 있지만 하나의 클래스만 상속받을 수 있다. 그리고 상속이 불가능한 값 타입(구조체, 열거형 등)은 여러 프로토콜 채택만 가능하다.

프로토콜의 기본구현(default implementations)은 다중상속과 매우 유사한 접근(approach)을 할만큼 유연성을 제공한다. 아래 코드를 보자.

```swift
protocol HelloPrinter {
    func sayHello()
}

extension HelloPrinter {
    func sayHello() {
        print("Hello")
    }
}
```

이제 이 프로토콜을 채택하는 타입을 새로 생성하면 해당 타입은 `sayHello`를 사용할 수 있다.

```swift
struct MyStruct: HelloPrinter {}

let myStruct = MyStruct()
myStruct.print() // Prints "Hello"
```

하지만 기본 구현이 되어있는 하나의 프로토콜을 채택하는 것으로 다중상속을 충분히 구현했다고 말하기는 어렵다. 더 중요한 것은 프로토콜이 `mixin` 개념을 충족하는 것이다.

### Understanding Mixins

mixin은 다른 클래스의 부모클래스가 되지 않으면서 다른 클래스들에 의해 사용될 메소드를 갖고 있는 클래스를 의미한다.

mixin의 아이디어는 간단하다 : pre-determining 과정 없이 extension을 정의하고자 하는 것이다. 이것은 부모 클래스가 추후에 결정되도록 하면서 서브 클래스를 특정(specify)하는 과정과 동일하다.

mixin은 스스로 인스턴스화 되어 사용되도록 설계되지 않는다. 이들은 다른 타입에 의해 더해질 행동(behavior)을 제공할 뿐이다. mixin의 주요 포인트는 아래와 같다.

- 행동(behavior)과 상태(state)를 모두 가질 수 있다.
- 초기화되지 않아야한다.
- 기능적으로 매우 구체화되어있다.
- 다른 mixins에 의해 서브클래싱 되지 않도록 한다.

이런 mixin의 도움으로 Swift에서 다중상속을 매우 유사하게 구현할 수 있다.

### Implementing a Mixin in Swift

`UIView`에 시각적 효과를 적용하는 것은 흔히 있는 일이다. 이런 과정에서 다중상속을 보이기 위해 아래 예제를 보자.

```swift
// MARK: - Blinkable

protocol Blinkable {
    func blink()
}

extension Blinkable where Self: UIView {
    func blink() {
        alpha = 1

        UIView.animate(
            withDuration: 0.5,
            delay: 0.25,
            options: [.repeat, .autoreverse],
            animations: {
                self.alpha = 0
        })
    }
}

// MARK: - Scalable

protocol Scalable {
    func scale()
}

extension Scalable where Self: UIView {
    func scale() {
        transform = .identity

        UIView.animate(
            withDuration: 0.5,
            delay: 0.25,
            options: [.repeat, .autoreverse],
            animations: {
                self.transform = CGAffineTransform(scaleX: 1.5, y: 1.5)
        })
    }
}

// MARK: - CornersRoundable

protocol CornersRoundable {
    func roundCorners()
}

extension CornersRoundable where Self: UIView {
    func roundCorners() {
        layer.cornerRadius = bounds.width * 0.1
        layer.masksToBounds = true
    }
}
```

그리고 UIView가 이 세 프로토콜을 채택하도록 한다.

```swift
extension UIView: Scalable, Blinkable, CornersRoundable {}
```

각 view와 그것들의 서브클래스는 mixins로부터 메소드를 공짜로(?) 사용할 수 있다.

```swift
aView.blink()
aView.scale()
aView.roundCorners()
```

결과는 아래와 같다.

![1](https://user-images.githubusercontent.com/35067611/108671663-254ce180-7524-11eb-9b63-5e3044ab2c1b.gif)

### The Diamond Problem

`Diamond Problem`은 아래 다이어그램으로 가장 잘 설명된다.

![1](https://user-images.githubusercontent.com/35067611/108671768-4a415480-7524-11eb-8142-ec036bed551a.png)

위 다이어그램에서 `MyClass`는 `Root`를 채택하는 `ChildA`, `ChildB` 프로토콜을 채택한다. `method()`는 Root에만 정의되어있지만 세 프로토콜 모두에서 extend 되어있다. 결과적으로 컴파일러는 어떤 method() 함수의 기본구현을 MyClass가 상속받아야할지 알 수 없게된다. 이런 관계를 코드로 나타내면 아래와 같다.

```swift
protocol Root {
    func method()
}

extension Root {
    func method() {
        print("Method from Root")
    }
}

protocol ChildA: Root {}

extension ChildA {
    func method() {
        print("Method from ChildA")
    }
}

protocol ChildB: Root {}

extension ChildB {
    func method() {
        print("Method from ChildB")
    }
}

class MyClass: ChildA, ChildB {} // Error: Type 'MyClass' does not conform to protocol 'Root'
```

이런 상황이 Diamond Problem이다. 또 다른 (더 짧은 버전의) 상황 또한 발생할 수 있다. 아래 사진을 보자.

![2](https://user-images.githubusercontent.com/35067611/108671775-4d3c4500-7524-11eb-9303-66ae3d24dc88.png)

위 다이어그램에서의 이슈는 아래 코드처럼 나타낼 수 있다.

```swift
protocol ProtocolA {
    func method()
}

extension ProtocolA {
    func method() {
        print("Method from ProtocolA")
    }
}

protocol ProtocolB {
    func method()
}

extension ProtocolB {
    func method() {
        print("Method from ProtocolB")
    }
}

class MyClass: ProtocolA, ProtocolB {} // Error: Type 'MyClass' does not conform to protocol 'ProtocolA'
```

ProtocolA, ProtocolB 모두 method() 함수의 기본구현을 갖고있는 위와 같은 경우는 `name collision`으로 이어진다. 컴파일러가 MyClass가 어떤 method() 함수를 상속받아야하는지 판단할 수 없는 것이다.

즉, diamond problem은 클래스나 값 타입이 상속 그래프에서 여러 경로로 프로토콜을 채택할 때 발생한다.

### Solution to Diamond Problem

아쉽게도 Swift에서 이 문제를 완벽하게 해결할 방법은 없다. 두가지 가능성이 존재하지만 각각의 단점이 있다.

- 충돌하는 메소드를 파생된 클래스에서 다시 구현하는 방법. 합성(composition)과 위임(delegation)으로 구현 가능하다. 문제는 코드 복잡도가 높아진다.
- 충돌하는 메소드를 조상들(Remote ancestors) 중 하나로 바꾼다. 이 방법의 문제점은 상속 그래프의 전체적인 그림을 알아야만 하는 전제조건이 있다.

### Wrapping Up

다중상속은 객체지향 프로그래밍의 기본적인 컨셉으로 올바르게 사용하면 매우 유용한 기능이다. Swift는 기본 언어 메커니즘으로 다중상속을 지원하지 않는다. 하지만 프로토콜에 기본구현을 갖게 함으로써 mixin 개념을 충족시키고 다중상속과 매우 유사하게 구현이 가능하다. mixins는 상속(inheritance)과 합성(composition)의 경계를 허문다. 이런 소프트웨어 디자인을 compositional inheritance라고 부른다.

## References

[Multiple Inheritance in Swift](https://www.vadimbulavin.com/multiple-inheritance-swift/)
