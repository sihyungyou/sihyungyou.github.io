---
layout: post
title: "iOS) [번역] Swift에서 race condition 해결하기"
tags: [iOS, Swift]
comments: true
---

> Teaching Core Data to dance with threads  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

[이전포스팅](https://sihyungyou.github.io/iOS-coredata-multithreaded/)에서 멀티스레딩 환경에서의 코어데이터 사용 시 어떻게 충돌을 방지할 수 있는지 살펴보았다. 그러다가 Swift 언어 전체적으로 봤을 때 어떻게 race condition을 해결할 수 있는지 궁금해졌다. 실제로 그동안 멀티 스레딩이 필요할 때마다 GCD가 잘 처리를 해줬기에 이 외에 다른 해결방법에 대해 생각해본 적이 없었다. 이번에도 [좋은 글](https://www.swiftbysundell.com/articles/avoiding-race-conditions-in-swift/)을 발견했고 번역하며 학습한 내용을 정리해보려한다.  

---

race condition은 예측 가능한 작업(operations) 시퀀스의 completion 순서가 예상하지 못하게 흘러가 프로그램 로직이 undefined state가 되는 경우를 말한다. 예를 들어 콘텐츠가 완전히 로드 되기 전에 UI가 업데이트 된다거나 로그인이 되지 않았는데 로그인 되었을 때의 화면을 보여주는 등의 경우가 있다.

race condition은 random하게 발생하기도 해, 디버깅이 매우 어렵다. random하다는 것은 버그를 다시 재현하는 데에 명확한 과정이 없다는 것을 의미하기 때문이다.

### An unpredictable race

예제를 먼저 보자. `AccessTokenService` 클래스는 토큰에 접근하여 네트워크 요청에 인증할 수 있도록 해주는 클래스다. 실질적인 네트워킹을 담당하는 `AccessTokenLoader`를 초기화하고 만약 토큰이 있다면 캐싱하여 불필요한 네트워크 요청이 발생하지 않도록 한다.

```swift
class AccessTokenService {
    typealias Handler = (Result<AccessToken>) -> Void

    private let loader: AccessTokenLoader
    private var token: AccessToken?

    init(loader: AccessTokenLoader) {
        self.loader = loader
    }

    func retrieveToken(then handler: @escaping Handler) {
        // If we have a cached token that is still valid, simply
        // return that directly as the result.
        if let token = token, token.isValid {
            return handler(.value(token))
        }

        loader.load { [weak self] result in
            // Cache the loaded token, then pass the result
            // along to the given handler.
            self?.token = result.value
            handler(result)
        }
    }
}
```

코드를 쭉 보면 매우 straightforward하다. 하지만 구현을 조금 더 자세히 살펴보면 `retrieveToken` 메소드가 두 번 불리는데 만약 첫번째 `load` 과정이 끝나지 않은 채 메소드가 다시 호출된다면 실제로는 두 개의 토큰을 가져오게된다. 이는 한 번에 하나의 인증 토큰만 유효해야 하는 인증과정에서 문제를 일으킨다. 이로써 두번째로 가져온 토큰이 첫번째 요청의 결과로 얻은 토큰을 무효화(invalidating) 시키는 race condition에 빠질 수 있다.

### Enqueueing pending handlers

그렇다면 이와 같은 race condition을 어떻게 예방할 수 있을까? 먼저 중복된 요청이 병렬적으로 수행되지 않도록 하고 retrieveToken에 전달된 handler들을 로딩중이라면 큐에 넣어놓을 수 있을 것이다.

그렇다면 이를 구현하기 위해 기존의 코드에 `pendingHandlers` 배열을 추가해보자. 그리고 retrieveTokens 메소드가 호출될 때마다 이 배열에 전달받은 handler를 추가한다. 그렇다면 배열에 하나의 handler만 존재하는지 확인함으로써 한번에 하나의 요청만 수행하도록 할 수 있다. 그리고 `handle` 메소드를 통해 enqueue 해놓은 handler들을 모두 실행시킨다.

```swift
class AccessTokenService {
    typealias Handler = (Result<AccessToken>) -> Void

    private let loader: AccessTokenLoader
    private var token: AccessToken?
    // We'll keep track of all enqueued, pending handlers using
    // a simple array.
    private var pendingHandlers = [Handler]()

    func retrieveToken(then handler: @escaping Handler) {
        if let token = token, token.isValid {
            return handler(.value(token))
        }

        pendingHandlers.append(handler)

        // We'll only start loading if the current handler is
        // alone in the array after being inserted.
        guard pendingHandlers.count == 1 else {
            return
        }

        loader.load { [weak self] result in
            self?.handle(result)
        }
    }
}

private extension AccessTokenService {
    func handle(_ result: Result<AccessToken>) {
        token = result.value

        let handlers = pendingHandlers
        pendingHandlers = []
        handlers.forEach { $0(result) }
    }
}
```

굳이 handler들을 클로저 내부에서 실행시키지 않고 handle 메소드를 별도로 구현한 이유는 여러 self 참조들이 필요해지기 때문이다. 흔히 사용하는 guard let 구문을 사용하기 보다는 단순하게 handle 메소드를 정의하여 평소와 같이 모든 handler들에 접근할 수 있도록 코드를 단순화했다.

코드를 이렇게 개선하여 retrieveToken 메소드가 한 시퀀스에서 여러번 불리더라도 하나의 토큰만 로드되고 여러 요청에 전달된 handler들은 순서대로 실행된다는 것을 보장할 수 있다.

위와 같이 비동기 completion handler를 큐에 넣는 방법을 통해 하나의 상태(single source of state)를 다룰 때 race condition을 방지할 수 있다. 하지만 이 방법에도 여전히 문제점이 있다.

### Thread safety

프로그래머가 작성하는 코드의 대부분은 사실 스레드 안정적이지 못하기에 멀티 스레딩 말고도 race condition을 유발하는 이유는 여럿 있다. UIKit은 메인 스레드에서만 돌아가므로 일반적으로 view layer와 관련한 대부분의 로직이 메인 스레드에서 불린다. 하지만 프로그램의 핵심 로직으로 들어가다보면 항상 메인 스레드에서 작업을 하지는 않는다.

만약 프로그램의 코드가 하나의 동일한 스레드에서 실행된다면 객체의 프로퍼티를 읽고 쓰는 작업은 항상 옳은 결과를 가질 것이다. 하지만 멀티 스레딩 환경이라면 두 스레드가 같은 프로퍼티를 동시에 읽거나 쓰는 상황이 발생하고 이는 한 스레드의 데이터가 즉각적으로 오래된(outdated) 데이터가 되도록 한다.

예를 들어 AccessTokenService 객체가 싱글 스레드에서 사용된다면 completion handler를 enqueueing 하는 방법으로도 충분할 것이다. 하지만 여러 스레드가 동일한 객체에 접근한다면 pendingHandlers 배열이 동시에 여러 스레드에 의해 내부 데이터가 변경될 것이다. 결국 다시 race condition 문제로 돌아온 것이다.

멀테 스레딩을 하면서 race condition을 해결하는 방법은 많지만 한가지 가장 명확한 방법은 애플이 제공하는 GCD API를 사용하는 것이다. GCD는 queue-based 추상화 방법을 통해 훨씬 쉽게 스레드를 다룰 수 있도록 도와준다.

다시 AccessTokenService로 돌아가서 `GCD`를 이용해 thread-safe 하도록 개선시켜보자. 먼저 내부 상태(internal state)를 동기화하기 위해 `DispatchQueue`를 사용할 것이다. 토큰 서비스 객체에서 DispatchQueue를 직접 초기화하거나 외부에서 주입받는 방식을 통해 클래스 내부에 큐를 둘 것이고 비동기 클로저들을(handler) 이 큐에 넣을 것이다.

```swift
class AccessTokenService {
    typealias Handler = (Result<AccessToken>) -> Void

    private let loader: AccessTokenLoader
    private let queue: DispatchQueue
    private var token: AccessToken?
    private var pendingHandlers = [Handler]()

    init(loader: AccessTokenLoader,
         queue: DispatchQueue = .init(label: "AccessToken")) {
        self.loader = loader
        self.queue = queue
    }

    func retrieveToken(then handler: @escaping Handler) {
        queue.async { [weak self] in
            self?.performRetrieval(with: handler)
        }
    }
}

private extension AccessTokenService {
    func performRetrieval(with handler: @escaping Handler) {
        if let token = token, token.isValid {
            return handler(.value(token))
        }

        pendingHandlers.append(handler)

        guard pendingHandlers.count == 1 else {
            return
        }

        loader.load { [weak self] result in
            // Whenever we are mutating our class' internal
            // state, we always dispatch onto our queue. That
            // way, we can be sure that no concurrent mutations
            // will occur.
            self?.queue.async {
                self?.handle(result)
            }
        }
    }
}
```

이전과 동일하게, 비동기 클로저 내부에서는 여러 self 참조를 피하기 위해 단순하게 `performRetrieval` 메소드로 handler를 실행시키도록 했다.

이로써 이제 AccessTokenService를 어떤 스레드에서든 사용할 수 있게 되었고 동시에 모든 로직이 예측가능해졌다(predictable) 또한 handler들은 여전히 순서대로 호출될 것이다.

### Conclusion

race condition을 완벽히 피하는 방법은 없지만 queuing이나 GCD를 사용하는 등의 테크닉을 통해 훨씬 안전한 코드를 작성할 수 있다. 비동기 코드를 작성할 때 해당 코드들이 동시에(concurrently) 호출되었을 때 어떻게 동작할지에 대해 생각해보는 것은 매우 도움이 된다.

그렇다면 우리는 항상 thread-safe하게 모든 코드를 작성해야할까? 그렇지 않다고 생각한다. 방금 본 예제와 달리 대부분의 코드는 메인 스레드 밖에서 사용될 일이 거의 없기 때문이다. 그러므로 완벽한 스레드 안정성을 모든 코드에 적용시키고자 하는 것은 over-engineering이 될 수 있다.

## References

[Avoiding race conditions in Swift](https://www.swiftbysundell.com/articles/avoiding-race-conditions-in-swift/)
