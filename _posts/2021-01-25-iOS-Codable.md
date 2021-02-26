---
layout: post
title: "iOS) Swift Codable 알아보기"
tags: [iOS, Swift]
comments: true
---

> Codable 프로토콜 정리  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

오늘은 손쉽게 JSON 인코딩/디코딩 과정을 처리해주는 `Codable` 프로토콜에 대해 알아보자.

## Codable 프로토콜이란?

먼저 공식문서에서 Codable 프로토콜을 어떻게 정의하는지 보자. 사실 Codable은 두 프로토콜을 함께 채택하는 것을 의미하는 Type alias로, Codable을 채택한다는 것은 결국 이 두 프로토콜을 채택하는 것이다.

```swift
typealias Codable = Decodable & Encodable
```

## Codable이 필요한 이유

그렇다면 "Codable이 필요한 이유"는 결국 "Decodable, Encodable 프로토콜을 채택해야하는 이유"라고 말할 수도 있겠다. 이에 대해 애플은 "Make your data types encodable and decodable for compatibility with external representations such as JSON." 라고 설명한다. 즉 데이터가 외부에서 사용되는 형태로(가장 일반적으로 JSON 형태) encode, decode 돼야 하는 상황을 위해 필요한 것이다.

실제로 프로그래밍을 하다보면 네트워크 통신으로 데이터를 주고받거나 디스크에 데이터를 저장하거나 API에 데이터를 submit 해야하는 상황을 자주 만나게된다. 이런 작업들은 데이터가 encode/decode 되도록 요구한다. 그리고 이 기능을 하는 것이 바로 Encodable, Decodable 프로토콜이다.

## 사용법

그렇다면 실제 사용 예제를 보면서 문법적으로 익혀보자. 참고로 String, Int, Double같은 standard library 타입들이나 Date, Data, URL같은 Foundation 타입들은 이미 Codable한 타입들이다. 하지만 개발자가 새롭게 정의하는 자료구조(구조체나 클래스)는 그렇지 않다.

```swift
struct Landmark: Codable {
    var name: String
    var foundingYear: Int

    // Landmark now supports the Codable methods init(from:) and encode(to:),
    // even though they aren't written as part of its declaration.
}
```

위와 같이 own type에 Codable을 채택함으로써 이 구조체에 대해서도 built-in 데이터타입으로 serialize 가능하게 되었다. 예를 들면 Landmark 구조체는 property list나 JSON을 handle하는 코드 없이도 Property ListEncoder와 JSONEncoder 클래스를 사용해 encode될 수 있는 것이다.

또한, 아래 코드와 같이 구조체 내의 구조체를 정의하는 방법으로 하위 구조까지 모두 Codable이 가능하다.

```swift
struct Coordinate: Codable {
    var latitude: Double
    var longitude: Double
}

struct Landmark: Codable {
    // Double, String, and Int all conform to Codable.
    var name: String
    var foundingYear: Int

    // Adding a property of a custom Codable type maintains overall Codable conformance.
    var location: Coordinate
}
```

Array, Dictionary, Optional과 같은 built-in 타입들도 codable한 타입을 포함하는 경우에 Codable 프로토콜을 채택한다. 아래 코드는 여러 built-in codable 타입을 포함하는 구조체의 경우에도 자동으로 Codable 프로토콜 conformance가 적용된다는 것을 보여준다.

```swift
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    var location: Coordinate

    // Landmark is still codable after adding these properties.
    var vantagePoints: [Coordinate]
    var metadata: [String: String]
    var website: URL?
}
```

지금까지 알아본 예시코드들은 모두 Codable 프로토콜을 채택하여 양방향(bidirectional)으로 encoding, decoding이 가능하도록했다. 하지만 네트워크 API를 호출하기만 하여 decode 과정이 필요없는 경우이거나 혹은 반대로 encoding이 필요없는 경우도 있다. 이런 경우 아래 코드처럼 작성하여 exclusive하게 encode나 decode를 할 수도 있다.

```swift
struct Landmark: Encodable {
    var name: String
    var foundingYear: Int
}

struct Landmark: Decodable {
    var name: String
    var foundingYear: Int
}
```

## CodingKeys

Codable 타입은 CodingKey 프로토콜을 채택하여 `CodingKeys`라는 특별한 중첩 열거형을 선언할 수 있다. 이 열거형이 선언됐다면 내부에 들어있는 case들은 곧 authoritative list or properties로써 codable 되는 타입의 인스턴스에 반드시 포함되어야함을 의미한다. 그러므로 encoding/decoding 과정에서 사용하지 않는 property가 있다면 CodingKeys 열거형에서 빼야 하고 default value가 필요하다.

만약 serialized data 포맷의 key가 정의한 구조체의 property와 맞지 않는다면 alternative key(대체키)를 CodingKeys 열거형에 제공해야한다. 이게 무슨 말인가 싶을 수 있는데 아래 코드를 보면 이해가 될 것이다.

```swift
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    var location: Coordinate
    var vantagePoints: [Coordinate]

    enum CodingKeys: String, CodingKey {
        case name = "title"
        case foundingYear = "founding_date"

        case location
        case vantagePoints
    }
}
```

CodingKeys 열거형에서 raw value로 사용되는 string 값들이 encoding, decoding 과정에서 사용되는 키값이다. 위 예제의 경우 name, foundingYear 두 property에 대해 alternative key를 정의하여 encoding, decoding 시 사용하게된다.

## 특정 Key, Value가 없는 경우

여기부터는 민소네님의 블로그를 참고하여 직접(manually) encode, decode 해야하는 상황과 그 과정에 대해서 정리해보았다. 애플 공식문서에도 마지막 부분에 이런 상황에 대해서 설명하긴 하는데 민소네님이 더 쉽고 직관적인 예시를 들어주신 것 같다(?)

아래와 같이 특정 키에 대해 변경사항이 있다고 생각해보자.

```swift
// before
{
    "a": "aa",
    "b": "bb"
}

// after
{
    "a": "aa"
}
```

당연히 변경사항을 적용하지 않고 이전과 동일하게 decode 시도를 하면 `keyNotFound` 에러가 발생할 것이다. 이 때 아래와 같이 JSON을 직접 decode 할 수 있다.

```swift
struct Sample1: Codable {
    var a: String
    var y: String

    enum CodingKeys: String, CodingKey {
        case a, y = "b"
    }

    init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        a = (try? values.decode(String.self, forKey: .a)) ?? ""
        y = (try? values.decode(String.self, forKey: .y)) ?? ""
    }
}
```

JSON을 decoding 할 때 int(from decode: Decoder)를 호출하여 해당부분에 직접 코드를 작성하는 것이다. 앞에서 문제가 됐던 keyNotFound는 기본 값을 넣어주는 방식으로 해결할 수 있다. 물론 모든 변수를 옵셔널 타입으로 처리하는 것도 가능하다. 이렇게 할 경우 key를 못찾는 변수들에 대해서 nil값으로 초기화될 것이다. (하지만 단순히 key를 못찾는 것이 아니라 JSON 데이터 자체가 null을 가질 수 있는 형태의 데이터라면 반드시 옵셔널로 선언을 해놓아야한다)

## 세 줄 요약

- Codable은 Encodable, Decodable을 모두 채택하는 Type alias로, JSON과 같은 외부에서 사용되는 형태의 데이터 포맷으로 encode/decode 해야할 때 필요하다.
- CodingKeys 열거형을 통해 key 이름과 타입의 이름을 다르게 정의할 수 있다.
- 데이터에 변동이 있는 경우를 대비해 직접 init(from decoder:)와 같은 메소드를 통해 decode 과정에 개입할 수 있다.

## References

[애플공식문서 - Codable](https://developer.apple.com/documentation/swift/codable)

[애플공식문서 - encoding and decoding](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types)

[민소네님 블로그 - [Swift4]Codable, 현실의 Codable 그리고 Extension](http://minsone.github.io/programming/swift-codable-and-exceptions-extension)
