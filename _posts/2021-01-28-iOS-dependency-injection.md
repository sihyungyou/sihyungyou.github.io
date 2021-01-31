---
layout: post
title: "iOS) DI(Dependency Injection) in Swift"
tags: [iOS, Swift]
comments: true
---

> 의존성 주입이란 무엇인가!  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

iOS 개발을 하다보면 Dependency Injection(의존성 주입)이라는 덕목을 갖춰야할 것 같은 느낌을 받는다. 프로그램의 결합도를 낮추고 테스트를 용이하게 하는 등 명확한 이점을 갖춘 의존성 주입은 왠지 모르게 "그냥 어려운 개념" 이라고 생각하게된다. 유튜브로 여러가지 Swift 관련 이야기들을 둘러보다가 나와 비슷한 생각을 하고 이에 대해 친절히 설명해준 영상을 발견했다. 이 영상에서는 의존성 주입에 대해 이렇게 말한다.

`Dependency injection is a 25-dollar term for 5-cent concept` - James Shore

(별 것 아닌 개념인데 너무 어렵게 받아들인다는, 상당히 비유가 맘에드는 문구다)

사실 의존성 주입은 instance variable을 주는 것 그 이상 그 이하도 아니다. 결국 객체에게 의존성을 말 그대로 주입시켜주는 것이다. 그래서 "의존성을 주입시키는 것"이 뭔데!라고 지금까지 나도 뜬구름 잡는 이야기처럼 들렸지만 결국 인스턴스 생성의 책임을 객체가 스스로 하도록 하지 않고 initializer, peroperty, method parameter의 형태로 "넘겨주는 것" 정도로 이해하면 될 것 같다. 아래 예시로 생각해보자.

```swift
class ViewController: UIViewController {
    var requestManager: RequestManager?
}
```

이런 경우 requesManager 변수를 세팅하는 방법에는 두 가지가 있을 수 있다. 먼저 ViewController로 하여금 직접 RequestManager 인스턴스를 생성하도록 하는 방법.

```swift
// ViewController 클래스 내부에서
var requestManager: RequestManager? = RequestManager()
```

의존성 주입을 적용시켜보자. 아래와 같이 코드를 작성할 수 있다. 말 그대로 인스턴스를 생성하는 책임을 ViewController에게 주는 것이 아니라 밖에서 생성하여 객체에게 "주입" 시키는 것이다.

```swift
// ViewController 클래스 외부의 코드
let viewController = ViewController()
viewController.requestManager = RequestManager()
```

이게 의존성 주입이 무엇인가?에 대한 대답의 끝이다. 여기서 끝내면 너무 무책임하니까.. 도대체 왜 의존성을 주입해야하는지, 그리고 주입하는 방법에는 어떤 것들이 있는지에 대해서도 살펴보자!

## Flexibility

아래 코드와 같이 프로토콜을 타입으로 하는 변수를 갖고 있다고 하자. 이 때 의존성 주입을 통해 외부에서 serializer 변수를 set 해주는 데에서 어떤 이득을 취할 수 있을까?

```swift
protocol Serializer {
    func serialize(data: Any) -> Data?
}

class RequestSerializer: Serializer {
    func serialize(data: Any) -> Data? {
        return nil
    }
}

class DataManager {
    var serializer: Serializer?
}
```

사실 이런 예시는 의존성 주입 자체의 장점이라기 보다는 프로토콜의 특성을 이해하고 결합하여 사용할 때 시너지 효과가 나는 경우다. DataManager 클래스에게 Serializer의 인스턴스를 생성하는 책임이 있다면 Serializer를 채택하는 RequestSerializer의 형태로만 생성할 수 있다.

```swift
var serializer: Serializer = RequestSerializer()
```

하지만 의존성 주입을 해준다면 상황에 따라 Serializer 프로토콜을 채택하는 어떤 객체 타입이든 serializer 변수로 대체할 수 있다. 즉, DataManager 객체는 이런 디테일에 대해 전혀 모르고 신경도 쓰지 않을 수 있다.

```swift
// 외부에서 주입시켜주는 경우
let dataManager = DataManager()
dataManager.serializer = RequestSerializer()
dataManager.serializer = OtherSerializer() // 이와 같이 필요에 맞게 다른 객체를 주입
```

## Transparency

의존성 주입을 해주면 클래스나 구조체의 책임이 더욱 명확해진다. 예를 들어 ViewController에 RequestManager를 주입시켜주어야 한다는 사실을 알고 있다면 우리는 ViewController가 어떤 객체에 의존성이 있고 역할을 하고 있는지 더욱 분명히 인지할 수 있다.

## Testing

유닛 테스팅 시, mock object를 사용하여 behavior를 분리할 수 있다는 점에서 굉장히 편리해진다. 예를 들어 네트워크 매니저 객체에 의존성을 갖는 객체를 테스팅 한다고 생각해보자. 그리고 유닛 테스트 시에는 네트워크 상황에 관계없이 테스트하기 위해 실제 통신을 하는 것이 아닌 통신이 성공했다고 가정하는 네트워크 매니저 객체를 사용해야 한다고 하자.

이런 경우 테스트코드에서는 통신은 하지 않지만 성공했을 경우의 결과를 넘겨주는 네트워크 매니저의 mock object를 외부에서 생성하여 주입해주고, 구현 코드에서는 실제 네트워크 매니저 객체를 생성해서 주입해주면 쉽게 테스트코드와 실제 기능을 하는 코드를 분리할 수 있다.

이번에는 의존성 주입이 코드 상에서 어떻게 이루어질 수 있는지 세 가지 방법에 대해 알아보자.

## Initializer Injection

말 그대로 initializer를 통해 외부에서 생성한 인스턴스를 주입시켜주는 것이다. 아래와 같이 작성할 수 있다.

```swift
class DataManager {
    private let serializer: Serializer

    init(serializer: Serializer) {
        self.serializer = serializer
    }
}
```

이 방법은 객체 생성 시 initializer를 통해 dependency를 파악할 수 있고, 주입받는 인스턴스를 immutable하게 유지할 수 있다는 장점이 있다. 또한 DataManager 인스턴스가 올바르게 configure 된다는 점을 보장할 수 있다.

## Property Injection

```swift
class DataManager {
    var serializer: Serializer?
}

// 외부 코드
let dataManager = DataManager()
dataManager.serializer = RequestSerializer()
```

위 코드와 같이 프로퍼티에 직접 접근하여 인스턴스를 대입시키는 방법이다. 매우 편리한 방법이지만 initializer inejction과 비교하면 immutable 속성을 유지할 수 없고 중간에 replace 될 수 있다는 것에 주의해야한다.

## Method Injection

```swift
class DataManager {
    func serializerRequest(_ request: Request, with serializer: Serializer) -> Data? {
        return nil
    }
}
```

serializer가 DataManager의 instance property가 아닌 경우 사용할 수 있는 방법이다. 이런 경우, DataManager 자체조차 serailizer에 대한 control을 갖고 있지 않다. 하지만 이런 방법은 여러 use case에 따라 매개변수를 달리하여 유연성에 집중할 수 있다는 장점이 있다.

## 세 줄 요약

- term이 어려워 보이지만 별 거 없으니 쫄지말자! instance를 넣어주는 것 그 이상 그 이하도 아니다.
- 유연성, 테스팅, 투명함 등의 장점을 갖고 있다.
- initializer, property, method injection 등 여러 방법이 있는데 상황에 맞게 사용하자.

## References

[Dependency Injection in Swift](https://youtu.be/-n8allUvhw8)
