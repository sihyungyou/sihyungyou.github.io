---
layout: post
title: "iOS) Swift의 클로저"
tags: [iOS, Swift]
comments: true
---

> 클로저 파헤쳐보기  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

## Definition

클로저란 무엇인가!?

Closures are self-contained blocks of functionality that can be passed around and used in your code.

self-contained 라는 표현이 좀 와닿지 않아 Language Guide 문서를 좀 더 읽어보았다. "shorter versions of function-like constructs without a full declaration and name" 이 표현으로 다시한번 설명해주는데 더 깔끔한 것 같다. 즉 이름이나 선언이 없어도 되는 함수같은 코드블럭 정도로 이해하면 될 것 같다.

![1](https://user-images.githubusercontent.com/35067611/104596260-1e29ec80-56b7-11eb-9b69-949da53c8ac0.png)

클로저를 사용하는 문법도 직관적이다. 전달받는 파라미터와 반환 타입을 명시해주고 수행해야 할 내용을 `in` 키워드로 구분하여 적어주면된다. 문법적으로 여러가지 기교(?)가 있다. 예를 들면 반환문 생략, 후행 클로저, 파라미터 생략 등. 물론 코드를 간략하게 쓰는것도 중요하겠지만 너무 줄여쓰는 것은 가독성을 해칠 수 있다. 이런 코딩스타일은 팀에서 허용한 범위 내에서 자유롭게 사용하면된다.

## Capturing Values

클로저를 본격적으로 다루기 이전에 글로만 봐서는 감이 당최 오지 않았던 변수캡처링 개념에 대해 다시 생각해보자.

A closure can capture constants and variables from the surrounding context in which it is defined. The closure can then refer to and modify the values of those constants and variables from within its body, even if the original scope that defined the constants and variables no longer exists.

위 글에서 주목할 점은 클로저 외부의 상수나 변수를 캡처한 후에 클로저 내부에서 값을 바꾸거나 가리킬 수 있다는 것이다. 이러한 성질때문에 다른 함수에 nested 되어있는 클로저만 떼어서 보면 선언조차 되어있지 않은 변수를 아무렇지않게 활용하고 있는 것을 볼 수 있다. 예를 들어 아래 코드를 보자.

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}
```

내부 클로저인 incrementer 함수 범위에서만 코드를 읽어보면 runningTotal, amount 라는 변수는 선언, 초기화도 되지 않았는데 갑자기 튀어나와서 사용되고있다. 이는 감싸고 있는 함수 makeIncrementer 범위에서 캡처된 것이다. 더 정확히 이야기하자면 `두 변수를 향한 참조`를 캡처한 것이다. 변수를 가리키는 참조를 캡처함으로써 makeIncrementer 함수가 끝나더라도 반환된 incrementer 함수 범위에서 두 변수가 사라지지 않고 사용될 수 있도록 하는것이다. 참고로 Swift에서는 최적화를 위해 항상 참조를 캡처하는 것이 아니라 클로저 내에서 해당 변수가 mutate 되는지 여부에 따라 변하지 않는다면 `값`을 복사해서 저장한다.

## 클로저는 참조타입!

위의 Capturing Values 주제로부터 두 가지 생각해볼 점이 생긴다. 클로저가 참조타입이라는 것과 메모리관리이다. 먼저 전자를 살펴보자.

위 예시를 그대로 이어가보면 아래와 같이 사용할 수 있다.

```swift
let incrementByTen = makeIncrementer(forIncrement: 10)

incrementByTen()
// returns a value of 10
incrementByTen()
// returns a value of 20
incrementByTen()
// returns a value of 30
```

makeIncrementer의 반환 타입은 특정한 값이 아니라 클로저다. 즉, incrementByTen 이라는 변수에는 (`let`으로 선언되었으니 정확히는 상수) incrementer() → Int 라는 클로저가 담긴다. 그리고 이 상수를 실행시킬 때 마다 클로저 내부의 값이 변한다.

incrementByTen은 분명 상수지만 가리키고 있는 클로저는 여전히 메모리에 살아있으며 그 내부의 runningTotal, amount 라는 변수도 내부에서 가리키고 있기에 이 변수들에 접근해서 값을 바꾸는 것 까지 가능하다. 이런 것이 가능한 이유는 Swift에서 함수와 클로저가 참조타입이기 때문이다. 상수나 변수에 함수, 클로저를 assign 하는 것은 사실 함수, 클로저를 가리키는 참조를 set 해주는 것과 같다.

## @escaping

클로저의 메모리 관리를 다루기에 앞서 매우 중요한 요소 중 하나인 escaping closures에 대해 먼저 알아보자! 클로저는 단순히 하나의 값을 갖는게 아니라 어떤 "행위"를 가질 수 있는 코드블럭단위이다. 이 행위가 꼭 클로저가 정의된 함수에서 실행되지 않고 클로저를 포함한 함수가 return 된 이후에 실행되도록 하고 싶을 때 escaping 키워드를 사용한다.

함수가 return 된 이후에 실행되도록 한다는 게 무슨 소리일까? 이번 주문앱 프로젝트에서 실제로 사용한 코드를 가져와봤다. HTTP GET 요청을 통해 JSON 데이터를 다운로드 받고 다운로드가 완료되면 JSON 데이터를 decoding 해서 반환해주는 함수다.

```swift
public func downloadJSONData(_ url: URL, completionHandler: @escaping (_ response: T?) -> Void) {
    let session = URLSession.shared

    session.dataTask(with: url) { data, response, error in
        guard error == nil else { return }

        if let data = data, let response = response as? HTTPURLResponse, response.statusCode == 200 {
            do {
                completionHandler(try JSONDecoder().decode(T.self, from: data))
            } catch {
                print(error.localizedDescription)
            }
        }
    }.resume()
}
```

여기서 핵심은 다운로드가 "**완료되면**" decoding 한다는 것이다. 네트워크 상태에 따라 HTTP 응답이 언제 올지도 모르는데 어떻게 이런 정확한 시점을 정해줄 수 있을까? 이 때 escaping 키워드를 사용해서 downloadJSONData 함수가 **return 된 이후에 decode를 시도**하도록 함수를 구현할 수 있다.

이 때 주의해야할 점이 있다. escaping 되는 클로저가 `self`를 캡쳐하고 self 대상이 클래스 인스턴스라면 순환참조에 빠질 수 있다. 왜 와이!? 클로저가 self를 가리키고 있다면 self에 정의된 함수 범위를 빠져나가도 여전히 캡쳐된 self가 메모리에서 사라지지 않기 때문이다.이런 이슈가 위에서 언급한 클로저와 메모리관리 측면에서 생각해볼 점이다.

## 클로저와 ARC, 캡쳐리스트

먼저 ARC에 대해 간단히 훑고 가도록 하자. ARC는 Automatic Reference Counting의 Swift가 메모리를 자동으로 관리해주기위해 필요한 친구다. Swift에서 deinit 되기위해서는 해당 메모리를 가리키는 모든 strong reference가 사라져야한다. 그러다보니 강한참조의 개수를 카운팅해서 메모리가 사라져도 되는지를 판단하는 것이다. 물론 이런 메커니즘을 이름에 써있듯 자동으로 해주지만 메모리 참조 순환에 대해서는 꼭 알아야한다. (ARC를 자세히 다루는 글도 나중에 써야겠다)

아무튼 클로저와 클래스 인스턴스가 서로 강한참조를 통해 가리키고 있다면 서로가 서로를 소유하고 있어 ARC를 0으로 만들지 못해 (ARC는 강한참조에 대해서만 카운팅을 올린다) 메모리에서 `deinit` 되지 못하는 상황이 발생한다 (memory leak)

![2](https://user-images.githubusercontent.com/35067611/104596267-1ff3b000-56b7-11eb-82c5-a9b620a7ca67.png)

이런 문제를 해결하기위해 클로저 내부에서 참조하는 것들을 강함참조가 아닌 `약한참조/미소유참조`가 되도록 명시해줄 수 있다.

```swift
class HTMLElement {
    let name: String
    let text: String?
    lazy var asHTML: () -> String = { [unowned self] in
        if let text = self.text {
            return "<\(self.name)>\(text)<\(self.name)>"
        } else {
            return "<\(self.name)>"
        }
    }
}
```

위에서 asHTML는 클로저로 만약 text가 init 된 상태라면 name-text-name을, 그렇지 않다면 name을 반환한다. 이 때 클로저 내부에서 self.name을 사용하면서 self(HTMLElement 클래스 인스턴스)를 참조하게 되는데 이 상황에서 그냥 강한참조로 둔다면 위 그림과 같은 상황이 발생한다. 그래서 unowned 키워드를 사용하여 미소유참조하여 ARC 카운팅이 0이 될 수 있도록 하는 것이다.

![3](https://user-images.githubusercontent.com/35067611/104596271-2124dd00-56b7-11eb-96ee-5a8cd6c6a6e2.png)

참고로 위 예시에서 unowned가 아니라 [weak self]를 사용해도 동일한 효과를 갖는다. 둘의 차이는 Optional type 여부에서 갈린다. 약한참조는 객체를 계속 추적하면서 객체가 사라지면 nil로 바꾸는 반면 unowned는 객체가 사라지면 댕글링 포인터가 남아서 crash가 날 위험이 있다. 즉 unowned 키워드는 절대 사라지지 않을 것이라고 보장되는 객체에만 설정해줘야한다. (그냥 weak을 쓰자)

## 언제 쓰이는가?

strong : 레퍼런스 카운트를 증가시켜 ARC로 인한 메모리 해제를 피하고 객체를 안전히 사용하고자 할 때

weak : retain cycle에 의한 memory leak 문제를 막기 위해

unowned : 객체의 라이프사이클이 명확하고 개발자에 의해 제어가능이 명확할 때

## Delegate 패턴에서 weak을 사용했던 이유

아.. 이번에 delegate 패턴을 열심히 연습해봤는데 왜 delegate 프로토콜변수가 weak으로 선언되어야했는지 이제서야 알게되었다.

```swift
nextViewController.delegate = self
```

일을 시키는 객체와 일을 하는 객체가 반드시 있는 상황에서 이렇게 delegate을 통해 객체를 연결시키면 두 ViewController가 서로를 소유하게 된다. 그래서 delegate 변수 선언 시 weak을 붙여 순환참조를 방지하는 것이었다!

```swift
weak var delegate: DelegateProtocol?
```

## 세 줄 요약

- 클로저는 코드의 묶음, 혹은 블럭으로 함수는 이름이 있는 클로저다.
- 클로저는 참조타입으로 사용 시 순환참조에 항상 주의해야한다.
- escaping 키워드를 통해 클로저가 실행되는 시점을 감싸는 함수의 return 이후로 정해줄 수 있다.

## References

[Closures - The Swift Programming Language (Swift 5.3)](https://docs.swift.org/swift-book/LanguageGuide/Closures.html)

[iOS 클로져 Closure 순환참조, 캡쳐리스트 활용 해결방법](https://0urtrees.tistory.com/72)

[[Swift] 메모리를 참조하는 방법 (Strong, Weak, Unowned)](https://devsrkim.tistory.com/entry/Swift-메모리를-참조하는-방법-Strong-Weak-Unowned)
