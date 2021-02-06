---
layout: post
title: "iOS) [번역] Managing a Shared Resource Using a Singleton"
tags: [iOS, Swift]
comments: true
---

> Cocoa Design Pattern - Singleton  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

# **Overview**

클래스의 전역적(globally) 접근, 공유 인스턴스(shared instance)를 제공하기 위해 싱글톤을 사용한다. 앱 내에서 소리를 재생하기 위한 오디오 채널이나 HTTP 요청을 하는 네트워크 매니저와 같은 리소스나 서비스에 대한 통일된 접근 경로 (unified access point)를 위해 싱글톤을 직접 생성할 수 있다.

# **Create a Singleton**

간단한 싱글톤을 static type property를 사용해 생성할 수 있다. 이는 여러 스레드에서 동시에 접근해도 lazily, 그리고 only once initialization을 보장한다.

```swift
class Singleton {
    static let sharedInstance = Singleton()
}
```

만약 초기화에 더하여 추가적인 setup이 필요하다면 클로저의 호출 결과를 global constant에 대입(assign) 할 수도 있다.

```swift
class Singleton {
    static let sharedInstance: Singleton = {
        let instance = Singleton()
        // setup code
        return instance
    }()
}
```

# References

[Apple Developer Documentation - Managing a Shared Resource Using a Singleton](https://developer.apple.com/documentation/swift/cocoa_design_patterns/managing_a_shared_resource_using_a_singleton)
