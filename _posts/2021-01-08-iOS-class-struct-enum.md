---
layout: post
title: "iOS) Struct vs. Class vs. Enum"
tags: [iOS, Swift]
comments: true
---

> 구조체 클래스 열거형 비교대조하기  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

구조체, 클래스, 열거형! 모두 비슷한 값, 속성들을 하나의 그룹으로 관리할 수 있도록 해주는 친구들이다. 이 셋의 공통점과 차이점을 비교대조 할 줄 알아야 하고 각각의 용도에 맞게 올바르게 코드를 짤 수 있어야한다. 물론 아주 대표적인 차이점들만 알아두려면 매우 간단하게 끝날 주제이다. 예를 들어 클래스는 참조타입이고 구조체와 열거형은 값타입이며 deinitializer, inheritance와 같이 클래스에만 지원되는 개념이 있고 열거형에는 stored property가 있을 수 없는 등등.. 하지만 오늘은 이 중에서도 "정말 구조체는 항상 값 타입일까?" 라는 질문에서 출발해 막연하게만 알고 있던 참조타입 vs. 값타입에 대해 조금 더 공부해보려한다.

## 참조타입 vs. 값타입

흔히 구조체, 클래스, 열거형의 공통점과 차이점을 떠올릴 때 가장 먼저 생각나는 것이 타입의 차이이다. 보통 구조체와 열거형은 값타입, 클래스는 참조타입이라고 하고, 이 차이점 때문에 셋 중에 어떤 것을 쓰느냐에 따라 프로그램이 작동하는 방식이 달라진다. 주로 동일한 클래스 인스턴스를 가리키는 여러 변수들이 있을 때 하나의 변수를 통해 클래스에 접근해서 값을 바꾸면 모든 변수들이 가리키는 하나의 인스턴스 내부 자체가 바뀌게 되고, 값 타입은 대입 과정에서 값의 복사가 일어나기 때문에 구조체 변수들이 모두 독립적인 객체를 갖게된다.

```swift
class Person {
	var name: String
	init(name: String) {
		self.name = name
	}
}

var p1 = Person(name: "Jason")
var p2 = p1

p1.name = "Daniel" // p2.name 또한 Daniel일 것
```

```swift
struct Person {
	var name: String
	init(name: String) {
		self.name = name
	}
}

var p1 = Person(name: "Jason")
var p2 = p1

p1.name = "Daniel" // p2.name은 여전히 Jason!
```

```swift
enum Language {
	case italian
	case english
}

var italian = Language.italian
let english = italian

italian = .english

print(italian) // .english
print(english) // .italian
```

가장 대표적인 참조타입과 값타입의 차이점이지만 여기서 주의해야할 것이 있다. 구조체는 항상 값타입인가? 라는 질문에는 "아니다"가 답이다. 구조체 내부에 참조타입 변수가 있을 수도 있고, 16 byte 이상의 크기를 가지면 구조체도 힙 영역에 생성됨으로써 참조타입이다. (JK님의 워딩을 빌리자면 무조건이라는 것은 없다) 또한 단순히 구조체만, 클래스만 사용하는 경우보다 둘을 섞어서 사용하는 경우가 훨씬 많다. 이런 경우에는 또 어떻게 작동하는지 볼 필요가 있다.

## Mixed Types

먼저 클래스 내에 구조체가 포함되는 경우에 대해 생각해보자.

```swift
struct Manufacturer {
    var name: String
}

class Device {
    var name: String
    var manufacturer: Manufacturer

    init(name: String, manufacturer: Manufacturer) {
        self.name = name
        self.manufacturer = manufacturer
    }
}

let apple = Manufacturer(name: "Apple")
// 여기서 iPhone, iPad 두 "클래스"는 동일하게 apple이라는 "구조체"를 속성으로 갖게된다
let iPhone = Device(name: "iPhone", manufacturer: apple)
let iPad = Device(name: "iPad", manufacturer: apple)

iPad.manufacturer.name = "Google"

print(iPhone.manufacturer.name)
print(iPad.manufacturer.name)
```

어떤 결과가 나올까? 두 클래스가 같은 구조체를 갖고있지만 출력결과는 각각 "Apple", "Google"로 나온다. 이는 꽤 직관적으로 이해가 된다. 애초에 구조체가 값 타입이니 iPhone, iPad라는 두 클래스가 init 될 때 값의 복사를 통해 속성을 저장했을 것이다.

그렇다면 구조체 내에 클래스가 포함되는 반대의 경우는 어떨까?

```swift
class Engine: CustomStringConvertible {
    var description: String {
        return "\(type) Engine"
    }

    var type: String

    init(type: String) {
        self.type = type
    }
}

struct Airplane {
    var engine: Engine
}

let jetEngine = Engine(type: "Jet")
// 여기서 jetEngine이라는 "참조타입변수"를 두 airplane 구조체가 독립적으로 복사해가느냐가 관건이다
let bigAirplane = Airplane(engine: jetEngine)
let littleAirplane = Airplane(engine: jetEngine)

littleAirplane.engine.type = "Rocket"

print(bigAirplane)
print(littleAirplane)
```

값타입의 구조체의 특성을 생각해보면 약간 이해가 가지 않는 결과를 보인다. 두 비행기 구조체 모두 엔진의 이름이 Rocket으로 바뀐 것이다. 참조타입을 따로 복사하여 초기화되는 것이 아니라 두 구조체가 하나의 Engine 인스턴스를 가리키는 변수를 가진 것이다.

이렇게 단순히 인스턴스를 가리킨다, 혹은 값을 복사한다 정도로만 알아두는 것에 더해서 구조체에 클래스가 들어있을 때와 그 반대의 경우까지 고려해보았다.

## Enums

이제 열거형에 대해서 이야기해보자! 열거형은 케이스를 나눌 때 정말 자주 사용하고 있으면서도 정확히 왜 꼭 열거형이어야 하는지에 대해서 확신이 없었다. 사실 같은 값타입인 구조체로도 열거형의 기능은 대부분 구현할 수 있을 것이다. 그런데 왜 굳이 열거형을 써야하는걸까? 몇 가지 장점들을 나열해보면 다음과 같다.

- 먼저 열거형은 `switch statement`와 만날 때 매우 강력해진다. 코드의 가독성이 높아지고, 간결해진다.
- 무엇보다 enum case을 사용하는 것의 장점은 컴파일 타임에 에러를 확인할 수 있다는 것이다. 배열이나 딕셔너리 등은 순서가 바뀌거나 nil값이 들어있는 등 런타임 에러가 발생할 여지가 있지만 열거형의 경우 이런 오류를 컴파일러가 잡아준다.
- 구조체로 열거형의 기능적인 요소는 대응할 수 있겠지만 구조체 내에 또 다른 구조체는 init하지 않으면 사용할 수 없다. 반면 열거형은 생성하지 않고 상수로 접근할 수 있도록 만들어져있기 때문에 실수의 여지가 줄어든다.
- 열거형의 각 case가 비트단위의 고유값을 갖기에 비교가 빠르다. 비교문에서의 장점도 갖는 것이다.

이렇게 좋은 사용성을 가진 열거형이지만 역시 단점도 있다. 열거형 타입을 너무 많이 사용하면 확장성 측면에서 불리하다. 열거형이 수정되면 모든 소스들을 매번 다시 컴파일해야하기 때문이다.

## 세 줄 요약

- 단순히 참조타입 vs. 값타입이라고만 알고 있는 것은 정확하지 않다. 항상 혼용해서 사용되는 경우에 대해서도 고려하자!
- 열거형은 케이스 비교에서 코드가 깔끔해질 뿐 아니라 컴파일 시점에 에러를 잡아주고 비트단위 고유값 비교를 통해 더 빠른 성능을 낼 수 있다는 장점이 있다.
- 무조건 클래스로 선언하지 말고 기능, 프로그램 구조, 확장성 등 여러 측면을 고려해서 클래스, 구조체, 열거형을 적절하게 활용하는 연습을 하자!

## References

[Swift: Value Types vs Reference Types, and When to Use Each](https://www.codementor.io/blog/value-vs-reference-6fm8x0syje)

[Swift (Enums VS Structures VS Classes)](https://saad-eloulladi.medium.com/swift-enums-vs-structures-vs-classes-938a4cd76c0d)
