---
layout: post
title: "iOS) [번역] Swift 면접 질문 - Intermediate"
tags: [iOS, Swift]
comments: true
---

> Swift Interview Questions and Answers - Intermediate  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

[Swift 면접 질문 - Beginner](https://sihyungyou.github.io/iOS-interview-questions-1-beginner/)에 이어지는 글로, 본 글은 Intermediate 레벨의 문답이다.

### Question #1

What's the difference between `nil` and `.none`?

### 나의 답

차이가 없다.

### 해설

Optional.none(줄여서 .none)과 nil이 같으므로 차이가 없다. 실제로 아래 구문은 true를 반환한다:

```swift
nil == .none
```

`nil`을 사용하는 것이 더 일반적이고 권장되는 컨벤션(convention)이다.

### Question #2

Here's a model of a thermometer as a class and a struct. The compiler will complain about the last line. Why does it fail to compile?

*Tip: Read the code carefully and think about it before testing it in a playground.

```swift
public class ThermometerClass {
    private(set) var temperature: Double = 0.0
    public func registerTemperature(_ temperature: Double) {
        self.temperature = temperature
    }
}

let thermometerClass = ThermometerClass()
thermometerClass.registerTemperature(56.0)

public struct ThermometerStruct {
    private(set) var temperature: Double = 0.0
    public mutating func registerTemperature(_ temperature: Double) {
        self.temperature = temperature
    }
}

let thermometerStruct = ThermometerStruct()
thermometerStruct.registerTemperature(56.0)
```

### 나의 답

구조체는 값 타입으로 `let`으로 선언할 경우 내부 프로퍼티가 변경가능(mutable)해도 변경할 수 없다. 위 코드에서 `registerTemperature` 메소드는 구조체 내부의 프로퍼티 값을 변경하는 기능을 하므로 let을 `var`로 바꾸어야한다.

### 해설

`ThermometerStruct`는 내부 변수를 변경하는 함수에 mutating이 명시되어 올바르게 선언되었다. 컴파일러가 에러를 내는 이유는 변경이 불가능하도록 let으로 생성된 인스턴스에 대해 registerTemperature를 호출했기 때문이다. let을 var로 바꾸면 에러가 사라질 것이다.

구조체에서 내부 상태(internal state)를 변경하는 메소드는 `mutating`으로 표시(mark) 해주어야한다. 하지만 변경 불가능한 변수(immutable variables)에서는 이렇게 표시된 메소드라도 호출(invoke)할 수 없다.

### Question #3

What will this code print and why?

```swift
var thing = "cars"

let closure = { [thing] in
    print("I love \(thing)")
}

thing = "airplanes"

closure()
```

### 나의 답

"I love cars" 를 출력한다. 그 이유는 closure가 정의되는(construct) 순간에 thing을 캡쳐했기 때문이다. 그 시점의 thing은 "cars"이다.

### 해설

"I love cars"를 출력한다. 캡쳐리스트는 클로저를 선언할 때 things의 복사본을 생성한다. 이는 캡쳐된 값은 thing에 새로운 값을 할당해도 변하지 않음을 의미한다. 만약 캡쳐리스트를 생략한다면 컴파일러가 복사 대신 참조를 사용할 것이다. 그러므로 클로저를 실행(invoke)할 때 변수의 변경사항을 반영할 것이다.

```swift
var thing = "cars"

let closure = {    
    print("I love \(thing)")
}

thing = "airplanes"

closure() // Prints: "I love airplanes"
```

### Question #4

Here's a global function that counts the number of unique values in an array:

```swift
func countUniques<T: Comparable>(_ array: Array<T>) -> Int {
    let sorted = array.sorted()
    let initial: (T?, Int) = (.none, 0)
    let reduced = sorted.reduce(initial) {
        ($1, $0.0 == $1 ? $0.1 : $0.1 + 1)
    }

    return reduced.1
}
```

It uses sorted, so it restricts T to types that conform to Comparable.

You call it like this:

```swift
countUniques([1, 2, 3, 3]) // result is 3
```

Rewrite this function as an extension method on Array so that you can write something like this:

```swift
[1, 2, 3, 3].countUniques() // should print 3
```

### 나의 답

```swift
extension Array where Element : Comparable {
    func countUniques() -> Int {
        let sorted = self.sorted()
        let initial: (Element?, Int) = (.none, 0)
        let reduced = sorted.reduce(initial) {
            ($1, $0.0 == $1 ? $0.1 : $0.1 + 1)
        }
        return reduced.1
    }
}
```

### 해설

countUniques 함수를 Array의 `extension`으로 작성할 수 있다.

```swift
extension Array where Element: Comparable {
    func countUniques() -> Int {
        let sortedValues = sorted()
        let initial: (Element?, Int) = (.none, 0)
        let reduced = sortedValues.reduce(initial) {
            ($1, $0.0 == $1 ? $0.1 : $0.1 + 1)
        }
        return reduced.1
    }
}
```

Element가 Comparable 프로토콜을 채택(conform)할 때에만 가능하다는 것에 주의하자.

### Question #5

Here's a function to divide two optional doubles. There are three preconditions to verify before performing the actual division:

- The dividend must contain a non `nil` value.
- The divisor must contain a non `nil` value.
- The divisor must not be zero.

```swift
func divide(_ dividend: Double?, by divisor: Double?) -> Double? {
    if dividend == nil {
        return nil
    }
    if divisor == nil {
        return nil
    }
    if divisor == 0 {
        return nil
    }

    return dividend! / divisor!
}
```

Improve this function by using the guard statement and without using forced unwrapping.

### 나의 답

```swift
func divide(_ dividend: Double?, by divisor: Double?) -> Double? {
    guard let dividend = dividend, let divisor = divisor, divisor != 0 else { return nil }

    return dividend / divisor
}
```

### 해설

아래와 같이 강제 언래핑 없이 안전한 코드를 작성할 수 있다.

```swift
func divide(_ dividend: Double?, by divisor: Double?) -> Double? {
    guard let dividend = dividend else { return nil }
    guard let divisor = divisor else { return nil }
    guard divisor != 0 else { return nil }
    return dividend / divisor
}
```

```swift
func divide(_ dividend: Double?, by divisor: Double?) -> Double? {
    guard let dividend = dividend, let divisor = divisor, divisor != 0 else {
        return nil
    }
    return dividend / divisor
}
```

### Question #6

Rewrite the method from question five using an if let statement.

### 나의 답

```swift
func divide(_ dividend: Double?, by divisor: Double?) -> Double? {
    if let dividend = dividend, let divisor = divisor, divisor != 0 {
        return dividend / divisor
    }

    return nil
}
```

### 해설

```swift
func divide(_ dividend: Double?, by divisor: Double?) -> Double? {
    if let dividend = dividend, let divisor = divisor, divisor != 0 {
        return dividend / divisor
    } else {
        return nil
    }
}
```

### Question #7

In Objective-C, you declare a constant like this:

```objectivec
const int number = 0;
```

Here is the Swift counterpart:

```swift
let number = 0
```

What are the differences between them?

### 나의 답

(Objective-C 코드를 공부해보지는 않았지만) Swift는 immutable 데이터 타입을 지원하여 const 변수가 아닌 상수 let, 변수 var로 구분하여 선언할 수 있게 되었다.

### 해설

`const`는 컴파일 타임에 반드시 해결되어야하는(must be resolved) 값(value)나 구문(expression)과 함께 컴파일 타임에 초기화되는 변수이다.

`let` 키워드로 생성된 immutable은 런타임에 결정되는 상수이다. 그러므로 static, dynamic expression과 함께 초기화가 가능하다. 아래와 같은 코드도 가능하다.

```swift
let higherNumber = number + 5
```

(단, 한번의 값 할당만 가능하다는 것에 주의할 것)

### Question #8

To declare a static property or function, you use the static modifier on value types. Here's an example for a structure:

```swift
struct Sun {
    static func illuminate() {}
}
```

For classes, it's possible to use either the `static` or the `class` modifier. They achieve the same goal, but in different ways. Can you explain how they differ?

### 나의 답

`class` 키워드가 붙는 클래스 메소드는 override가 가능하지만 `static`은 그렇지 않다.

### 해설

static 키워드는 프로퍼티나 함수를 static하게 만들지만 override 불가능하다. class 키워드는 동일한 효과를 갖지만 override가 가능하다. 즉, static은 `class final`과 동일하다.

```swift
class Star {
    class func spin() {}
        static func illuminate() {}
}

class Sun : Star {
    override class func spin() {
        super.spin()
    }
    // error: class method overrides a 'final' class method
    override static func illuminate() {
        super.illuminate()
    }
}
```

### Question #9

Can you add a stored property to a type by using an extension? How or why not?

### 나의 답

할 수 없다. 근데 왜인지는.. 답을보자!

### 해설

불가능하다. 존재하는 type에 새로운 행동(behavior)를 추가하기 위해 extension을 사용할 수 있다. 하지만 타입 자체나 타입의 인터페이스를 변경(alter)하는 것은 불가능하다. 만약 stored property를 추가한다면 새 값을 저장하기 위한 새로운 별도의 메모리(extra memory)가 필요하다. extension은 이런 작업을 할 수 없다.

### Question #10

What is a protocol in Swift?

### 나의 답

질문이 너무 광범위한데.. 프로토콜은 메소드와 프로퍼티 등을 선언하여 이를 채택하는 객체는 반드시 구현을 하도록 하는 일종의 청사진과 같은 역할을 한다.

### 해설

프로토콜은 메소드, 프로퍼티와 다른 요구사항에 대한 청사진을 정의하는 타입이다. 클래스, 구조체, 열거형은 프로토콜을 채택하여 요구사항을 구현할 수 있다.

프로토콜은 함수를 직접 구현하지 않고 정의만 한다. 프로토콜을 extend 하여 특정 요구사항에 대해 기본 구현(default implementation)이나 추가적인 기능(additional functionality)을 제공할 수 있다.  

## References  
[Swift Interview Questions and Answers](https://www.raywenderlich.com/762435-swift-interview-questions-and-answers#toc-anchor-013)
