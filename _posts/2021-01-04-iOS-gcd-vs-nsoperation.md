---
layout: post
title: "iOS) GCD vs. NSOperation"
tags: [iOS, Swift]
comments: true
---

> 애플의 멀티스레딩 API 뜯어보기  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

GCD는 멀티코어 환경에서 최적화된 프로그래밍을 할 수 있도록 애플이 지원하는 저수준 API이다. GCD는 C-based인 반면 NSOperation, NSOperationQueue는 Objective-C 클래스로, GCD를 Objective-C로 감싼 수준의 API다.

즉, NSOperation을 사용한다고해서 GCD를 배제한 것이 아닌 GCD도 사용하고 있는 것이다. 참고로 NSOperation클래스 자체는 유용한 작업을 수행하기 위해서 서브클래싱 되어야 하는 추상 기본 클래스로 사용하고 싶다면 상속을 받아서 사용해야한다.

## 차이점

- GCD는 비교적 가볍고 NSOperationQueue는 복잡하고 무거운 편이다.
- NSOperation은 대신 다른 장점들을 갖고 있는데 NSOperation에 대한 pause, cancel, resume이 가능하다.
- NSOperation은 두 작업(operations) 사이에 의존관계를 설정하여 다른 작업이 끝나고 true를 반환해야만 작업을 시작하는 등의 컨트롤이 가능하다.
- NSOperation은 작업(operation)의 상태를 ready, executing, finished로 구분하는데 이 여러 상태들을 모니터링 가능하다.
- NSOperation은 큐에 들어가 동시에 수행될 수 있는 최대 작업 개수를 정할 수 있다.

## 언제 무엇을 사용해야하는가?

위와 같은 특성들을 고려해서 큐에 있는 작업들에 대해 좀 더 다양하고 미세한 컨트롤이 필요하다면 NSOperation을, 적은 오버헤드로 단순히 작업들을 멀티스레드 환경에서 수행시키고 싶은 것이라면 GCD를 사용하는 것이 좋다.

## References

[What is the difference in GCD and NSOperation?](https://www.quora.com/What-is-the-difference-in-GCD-and-NSOperation)
