---
layout: post
title: "iOS) [번역] Swift 면접 질문 - Beginner"
tags: [iOS, Swift]
comments: true
---

> Swift Interview Questions and Answers - Beginner  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

[raywenderlich](https://www.raywenderlich.com/762435-swift-interview-questions-and-answers#toc-anchor-013)에서 Swift 인터뷰 질문들에 대한 문답을 Beginner, Intermediate, Advanced 단계로 나누어 정리해놓은 글을 발견했다. 각 레벨의 질문과 답에 대해 내 나름대로 답을 써보고, 영어로 제공되는 답을 번역해보려한다. 본 글은 Beginner 레벨이다.

### Question #1

```swift
struct Tutorial {
    var difficulty: Int = 1
}

var tutorial1 = Tutorial()
var tutorial2 = tutorial1
tutorial2.difficulty = 2
```

What are the values of `tutorial1.difficulty` and `tutorial2.difficulty`? Would this be any different if Tutorial was a class? Why or why not?

### 나의 답

tutorial1.difficulty는 1, tutorial2.dificulty = 2일 것이다. 그 이유는 Tutorial이 클래스가 아닌 구조체이기 때문. 클래스라면 두 인스턴스의 difficulty 모두 업데이트된 2를 값으로 가질 것이다.

### 해설

tutofial1.difficulty = 1, tutorial2.difficulty = 2이다. 구조체는 Swift의 값 타입이다. 참조가 아닌 값을 복사한다. 그러므로 아래의 코드는 tutorial1의 복사본을 tutorial2에 대입(assign) 하는 코드일 것이다.

```swift
var tutorial2 = tutorial1
```

그러므로 tutorial2의 변경사항이 tutorial1에 반영되지 않는다.

만약 Tutorial이 클래스였다면 tutorial1.difficulty, tutorial2.difficulty 모두 2일 것이다. 클래스는 Swift의 참조타입으로 tutorial1의 프로퍼티를 변경하면 tutorial2에도 영향을 미치고 역도 동일하다.

### Question #2

You’ve declared view1 with var, and you’ve declared view2 with let. What’s the difference, and will the last line compile?

```swift
import UIKit

var view1 = UIView()
view1.alpha = 0.5

let view2 = UIView()
view2.alpha = 0.5 // Will this line compile?
```

### 나의 답

UIView는 클래스타입이므로 let으로 선언해도 내부 프로퍼티를 변경할 수 있다. 그러므로 컴파일에 문제가 없을 것이다.

### 해설

마지막 줄은 컴파일이 된다. `view1`은 변수이고 `UIView`의 인스턴스를 재할당(reassign)할 수 있다. `let` 키워드로는 한번만 할당(assign)이 가능하므로 아래의 코드는 컴파일이 불가능하다.

```swift
view2 = view1 // Error: view2 is immutable
```

하지만 UIView는 클래스이며 참조 시멘틱(reference semantics)이므로 `view2`의 프로퍼티를 변경할 수 있다.

```swift
let view2 = UIView()
view2.alpha = 0.5 // Yes!
```

### Question #3

This complicated code sorts an array of names alphabetically. Simplify the closure as much as you can.

```swift
var animals = ["fish", "cat", "chicken", "dog"]
animals.sort { (one: String, two: String) -> Bool in
    return one < two
}
print(animals)
```

### 나의 답

```swift
animals.sort { return $0 < $1 }
```

### 해설

타입추론 시스템이 자동으로 클로저 내의 두 매개변수와 반환값의 타입을 계산할 것이다. 그러므로 해당 부분을 코드에서 제거할 수 있다.

```swift
animals.sort { (one, two) in return one < two }
```

또한, `$i` 표기(notation)로 매개변수의 이름을 대체(substitute) 할 수 있다.

```swift
animals.sort { return $0 < $1 }
```

`single statement` 클로저에서는 return 키워드를 생략할 수 있다. 마지막 구문(last statement)의 값이 클로저의 반환값이 된다.

```swift
animals.sort { $0 < $1 }
```

마지막으로, Swift는 배열의 원소들이 `Equatable` 프로토콜을 채택하는 것을 알고 있으므로 아래와 같이 간단하게 쓸 수 있다.

```swift
animals.sort(by: <)
```

### Question #4

This code creates two classes: Address and Person. It then creates two Person instances to represent Ray and Brian.

```swift
class Address {
    var fullAddress: String
    var city: String

    init(fullAddress: String, city: String) {
        self.fullAddress = fullAddress
        self.city = city
    }
}

class Person {
    var name: String
    var address: Address

    init(name: String, address: Address) {
        self.name = name
        self.address = address
    }
}

var headquarters = Address(fullAddress: "123 Tutorial Street", city: "Appletown")
var ray = Person(name: "Ray", address: headquarters)
var brian = Person(name: "Brian", address: headquarters)
```

Suppose Brian moves to the new building across the street; you'll want to update his record like this:

```swift
brian.address.fullAddress = "148 Tutorial Street"
```

This compiles and runs without error. **If you check the address of Ray now, he's also moved to the new building.**

```swift
print (ray.address.fullAddress)
```

What's going on here? How can you fix the problem?

### 나의 답

`Person` 클래스 내부의 `address` 프로퍼티 또한 클래스 타입임에 주목하자. `headquarters`라는 하나의 Address 클래스 인스턴스를 ray, brian 초기화 과정에서 넘겨주었다. 그러므로 두 인스턴스의 address 프로퍼티는 동일한 headquarters 인스턴스를 가리킨다. 그러므로 한쪽에서의 변경사항이 다른쪽에도 영향을 미치는 것이다.

### 해설

`Address`는 클래스이며 참조 시멘틱(reference semantics)이므로 `headquarters`는 ray나 brian 중 어떤 객체를 통해 접근해도 동일한 인스턴스이다. headquarters의 주소를 바꿈으로써 두 쪽 모두 바꾸게 될 것이다. 이를 해결하기 위한 방법은 brian을 위한 Address를 새로 생성하든지 Address를 구조체로 바꾸는 것이다.

### Question #5

What is an optional and which problem do optionals solve?

### 나의 답

옵셔널은 한 변수에 "값의 부재"를 허용할 수 있도록 하는 기능이다. 옵셔널을 통해 데이터의 값 자체 뿐만 아니라 데이터가 있고, 없음을 표현할 수 있다.

### 해설

옵셔널은 값으 부재(lack of value)를 허용한다. Objective-C에서는 값의 부재가 `nil`이라는 특별한 값을 이용하여 참조타입에만 허용됐었다. 값 타입의 경우 (`int`, `float`과 같은) 이런 것이 불가능했다.

Swift는 값의 부재를 참조와 값 타입 모두에 대해 확장한다. 옵셔널 변수는 특정 값이나 nil을 가질 수 있다.

### Question #6

Summarize the main differences between a structure and a class.

### 나의 답

구조체는 값 타입, 클래스는 참조타입이다. 클래스는 상속이 가능하지만 구조체는 불가능하다.

### 해설

- 클래스는 상속을 지원한다. 구조체는 불가능하다
- 클래스는 참조타입이다. 구조체는 값 타입이다.

### Question #7

What are generics and which problem do they solve?

### 나의 답

generics는 타입을 특정하지 않고 어떤 자료형이든 받아들일 수 있도록 한다. generics는 중복된 코드를 줄여주는 효과가 있다.

### 해설

Swift에서는 generics를 데이터(클래스, 구조체, 열거형)와 함수 모두에 대해 사용할 수 있다.

Generics는 코드 중복의 문제를 해결한다. 한 타입의 매개변수를 받는 메소드가 있을 때 일반적으로 다른 타입에 대해 동일한 메소드를 수행하기 위해서는 이 메소드를 복제하기 마련이다.

예를 들어 아래 코드에서 second function은 first function과 정수 대신 문자열을 매개변수로 받는 것을 제외하고 동일하다(clone)

```swift
func areIntEqual(_ x: Int, _ y: Int) -> Bool {
    return x == y
}

func areStringsEqual(_ x: String, _ y: String) -> Bool {
    return x == y
}

areStringsEqual("ray", "ray") // true
areIntEqual(1, 1) // true
```

generics를 사용(adopt)함으로써, 두 함수를 하나로 합칠치고 `type safety`를 지킬 수 있다.

```swift
func areTheyEqual<T: Equatable>(_ x: T, _ y: T) -> Bool {
    return x == y
}

areTheyEqual("ray", "ray")
areTheyEqual(1, 1)
```

이 경우에는 equality를 체크하는 메소드이기 때문에 매개변수로 전달되는 모든 타입은 Equatable 프로토콜을 구현해야한다.

### Question #8

In some cases, you can't avoid using implicitly unwrapped optionals. When? Why?

### 나의 답

음.. 피할 수 없나? 옵셔널 체이닝, 옵셔널 바인딩을 통해서 가능하지 않나?!

### 해설

implicitly unwrapped optionals를 사용하는 가장 일반적인 이유는 아래와 같다:

- instantiation time에 의해 nil이 아닌 프로퍼티를 초기화할 수 없을 때. 가장 대표적인 예로 자신의 owner보다 항상 늦게 초기화되는 Interface Builder outlet이 있다. 이런 경우(Interface Builder에서 올바르게 configure 되었다고 가정하는 경우)에는 outlet이 사용하기 전에 non-nil이라는 것을 보장할 수 있다.
- 두 인스턴스가 서로를 참조하고 서로 non-nil 참조여야 하는 순환참조(strong reference cycle) 문제를 해결할 때. 이런 경우엔 한쪽의 참조를 unowned로 선언하고 다른쪽은 implicitly unwrapped optional을 사용한다.

### Question #9

What are the various ways to unwrap an optional? How do they rate in terms of safety?

```swift
var x : String? = "Test"
```

Hint: There are seven ways.

### 나의 답

- 옵셔널 바인딩
- 옵셔널 체이닝
- 강제 언래핑
- implicitly unwrapping optional
- 나머지 세개는 모르겠..

### 해설

- 강제 언래핑 (unsafe)

    ```swift
    let a: String = x!
    ```

- implicitly unwrapped 변수 선언 (unsafe in many cases)

    ```swift
    var a = x!
    ```

- 옵셔널 바인딩 (safe)

    ```swift
    if let a = x {
      print("x was successfully unwrapped and is = \(a)")
    }
    ```

- 옵셔널 체이닝 (safe)

    ```swift
    let a = x?.count
    ```

- nil coalescing 연산 (safe)

    ```swift
    let a = x ?? ""
    ```

- guard문 (safe)

    ```swift
    guard let a = x else { return }
    ```

- optional 패턴 (safe)

    ```swift
    if case let a? = x {
        print(a)
    }
    ```

## References  
[Swift Interview Questions and Answers](https://www.raywenderlich.com/762435-swift-interview-questions-and-answers#toc-anchor-013)
