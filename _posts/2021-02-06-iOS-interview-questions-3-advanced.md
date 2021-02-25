---
layout: post
title: "iOS) [번역] Swift 면접 질문 - Advanced"
tags: [iOS, Swift]
comments: true
---

> Swift Interview Questions and Answers - Advanced  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

[Swift 면접 질문 - Intermediate](https://sihyungyou.github.io/iOS-interview-questions-2-intermediate/)에 이어지는 글로, 본 글은 Advanced 레벨의 문답이다.

### Question #1

Consider the following structure that models a thermometer:

```swift
public struct Thermometer {
    public var temperature: Double
    public init(temperature: Double) {
        self.temperature = temperature
    }
}
```

To create an instance, you can use this code:

```swift
var t: Thermometer = Thermometer(temperature:56.8)
```

But it would be nicer to initialize it this way:

```swift
var thermometer: Thermometer = 56.8
```

Can you? How?

### 나의 답

이게 돼..? 몰랐음

### 해설

Swift는 literal value로 타입을 초기화할 수 있도록 하는 프로토콜을 정의한다. 이런 프로토콜을 채택한 후, public initializer를 제공하면 특정 타입에 대해 literal initialization이 가능하다. 위 문제의 경우, 아래 코드와 같이 작성하면 된다.

```swift
extension Thermometer: ExpressibleByFloatLiteral {
    public init(floatLiteral value: FloatLiteralType) {
        self.init(temperature: value)
    }
}
```

### Question #2

Swift has a set of pre-defined operators to perform arithmetic or logic operations. It also allows the creation of custom operators, either unary or binary.

Define and implement a custom ***^^*** power operator with the following specifications:

- Takes two `Int`s as parameters.
- Returns the first parameter raised to the power of the second.
- Correctly evaluates the equation using the standard algebraic order of operations.
- Ignores the potential for overflow errors.

### 나의 답

음... 공부해보자

### 해설

custom operator는 두 단계를 거쳐 생성할 수 있다: declaration, implementation 이다.

declaration은 타입(unary or binary), operator를 구성하는 문자의 나열, 그리고 연관성(associativity)과 우선순위(precedence)를 특정하기 위해 `operator` 키워드를 사용한다. Swift 3.0은 precedence의 구현을 precedence group을 사용하도록 변경했다.

여기서 연산자는 `^^`이며 타입은 `infix`(binary)이다. associativity는 `right`이다; 다른 의미로 같은 우선순위를 갖는 ^^ 연산자는 오른쪽에서 왼쪽으로 evaluate되어야 한다는 뜻이다.

Swift에서 지수 연산자(exponential operations)에는 predefined된 표준 우선순위가 없다. 일반적으로 수학에서 사용되는 연산자 우선순위를 생각해보면 지수계산은 곱하기/나누기보다 먼저 계산되어야한다. 그러므로 우선순위를 곱셈보다 높게 설정해야한다.

```swift
precedencegroup ExponentPrecedence {
    higherThan: MultiplicationPrecedence
    associativity: right
}
infix operator ^^: ExponentPrecedence
```

구현은 아래와 같다.

```swift
func ^^(base: Int, exponent: Int) -> Int {
    let l = Double(base)
    let r = Double(exponent)
    let p = pow(l, r)
    return Int(p)
}
```

위 경우는 overflow에 대해 전혀 예외처리를 하지 않은 코드임에 주의하자. `Int.max`보다 큰 수처럼 Int로 표현할 수 없을만큼 큰 수가 결과로 계산되면 런타임에러가 발생할 것이다.

### Question #3

Consider the following code that defines Pizza as a struct and Pizzeria as a protocol with an extension that includes a default implementation for `makeMargherita()`:

```swift
struct Pizza {
    let ingredients: [String]
}

protocol Pizzeria {
    func makePizza(_ ingredients: [String]) -> Pizza
    func makeMargherita() -> Pizza
}

extension Pizzeria {
    func makeMargherita() -> Pizza {
        return makePizza(["tomato", "mozzarella"])
    }
}
```

You'll now define the restaurant Lombardi’s as follows:

```swift
struct Lombardis: Pizzeria {
    func makePizza(_ ingredients: [String]) -> Pizza {
        return Pizza(ingredients: ingredients)
    }

    func makeMargherita() -> Pizza {
        return makePizza(["tomato", "basil", "mozzarella"])
    }
}
```

The following code creates two instances of Lombardi's. Which of the two will make a margherita with basil?

```swift
let lombardis1: Pizzeria = Lombardis()
let lombardis2: Lombardis = Lombardis()

lombardis1.makeMargherita()
lombardis2.makeMargherita()
```

### 나의 답

두 인스턴스 모두 basil을 사용할 것이다.

### 해설

둘 다 사용한다. `Pizzeria` 프로토콜은 `makeMargherita()` 메소드를 선언(declare)하고 기본구현(default implementation)을 제공한다. `Lombardis` 구현은 이 디폴트 메소드를 override한다. 두 경우 모두 프로토콜에 메소드가 선언되어 있으므로 런타임에 올바른 구현을 호출하게된다.

만약 프로토콜에서 makeMargherita() 메소드를 선언하지 않고 extension에서 default implementation을 제공한다면 어떻게 될까?

```swift
protocol Pizzeria {
    func makePizza(_ ingredients: [String]) -> Pizza
}

extension Pizzeria {
    func makeMargherita() -> Pizza {
        return makePizza(["tomato", "mozzarella"])
    }
}
```

이 경우에는 `lombardis1`은 extension에 정의된 메소드를 사용하므로 `lombardis2` 객체만 basil을 사용할 것이다.

### Question #4

The following code has a compile time error. Can you spot it and explain why it happens? What are some ways you could fix it?

```swift
struct Kitten {
}

func showKitten(kitten: Kitten?) {
    guard let k = kitten else {
        print("There is no kitten")
    }
    print(k)
}
```

Hint: There are three ways to fix the error.

### 나의 답

guard문에 return이 없다?

### 해설

guard문의 else 블록은 `return`, `@noreturn` 호출, 혹은 excetption을 `throw`하는 방식 중 하나를 선택하여 exit path를 반드시 명시해야한다

먼저 return문의 활용이다.

```swift
func showKitten(kitten: Kitten?) {
    guard let k = kitten else {
        print("There is no kitten")
        return
    }
    print(k)
}
```

exception throw는 아래 코드와 같다.

```swift
enum KittenError: Error {
    case NoKitten
}

struct Kitten {
}

func showKitten(kitten: Kitten?) throws {
    guard let k = kitten else {
        print("There is no kitten")
        throw KittenError.NoKitten
    }
    print(k)
}

try showKitten(kitten: nil)
```

마지막으로 @noreturn 함수인 `fatalError()` 함수를 호출하는 방식이다.

```swift
struct Kitten {
}

func showKitten(kitten: Kitten?) {
    guard let k = kitten else {
        print("There is no kitten")
        fatalError()
    }
    print(k)
}
```

### Question #5

Are closures value or reference types?

### 나의 답

Swift에서 클로저는 참조타입이다.

### 해설

클로저는 참조타입이다. 클로저를 변수에 할당(assign) 하고 해당 변수를 다른 변수로 복사한다면, 동일한 클로저와 클로저의 캡쳐리스트를 향한 참조를 복사하게된다.

### Question #6

You use the UInt type to store unsigned integers. It implements the following initializer to convert from a signed integer:

```swift
init(_ value: Int)
```

However, the following code generates a compile time error exception if you provide a negative value:

```swift
let myNegative = UInt(-1)
```

An unsigned integer by definition cannot be negative. However, it's possible to use the memory representation of a negative number to translate to an unsigned integer. How can you convert an Int negative number into an UInt while keeping its memory representation?

### 나의 답

memory representation을 뭐라고 번역해야할지 모르겠다. 직접 값을 전달하는 것이 아니라 `&` 노테이션을 붙여 주소값을 전달하는 것 아닐까? (전혀 아니었다! ^_^)

### 해설

이런 경우를 위한 initializer가 있다.

```swift
UInt(bitPattern: Int)
```

그래서 아래와 같이 초기화할 수 있다.

```swift
let myNegative = UInt(bitPattern: -1)
```

### Question #7

Can you describe a circular reference in Swift? How can you solve it?

### 나의 답

순환참조는 두 객체가 서로를 강하게 참조하여 ARC가 메모리에서 해제할 수 없는 상황을 말한다. 둘 중 한 참조를 `weak`, 혹은 `unowned` 참조로 바꾸어 해결할 수 있다.

### 해설

순환참조는 두 인스턴스가 서로에 대해 강한참조를 가질 때 발생하며 두 인스턴스 중 어느것도 메모리에서 해제될 수 없기에 메모리 누수를 야기한다. 그 이유는 강한참조가 있을 때는 해당 인스턴스를 메모리에서 해제(deallocate)할 수 없는데 각 인스턴스가 다른 인스턴스를 메모리에 살아있도록 강하게 참조하고 있기 때문이다.

이 문제는 강한참조를 `weak`, `unowned` 참조로 대체함으로써 해결 가능하다.

### Question #8

Swift allows the creation of recursive enumerations. Here's an example of such an enumeration with a Node case that takes two associated value types, T and List:

```swift
enum List<T> {
    case node(T, List<T>)
}
```

This returns a compilation error. What is the missing keyword?

### 나의 답

전혀 모르겠다; 해설을 보고 공부해봐야겠다. (사실 recursive enumerations에 대해 처음 들어봄)

### 해설

recursive enumeration을 가능하도록 하는 `indirect` 키워드가 있어야한다.

```swift
enum List<T> {
    indirect case node(T, List<T>)
}
```

## References  
[Swift Interview Questions and Answers](https://www.raywenderlich.com/762435-swift-interview-questions-and-answers#toc-anchor-013)
