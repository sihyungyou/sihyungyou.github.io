---
layout: post
title: "iOS) Asynchronous Programming in iOS"
tags: [iOS, Swift]
comments: true
---

> iOS에서의 비동기 프로그래밍  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

Swift의 꽃!? 비동기 프로그래밍에 대해서 알아보자. 비동기는 앱 개발에 있어서 필수적이지만 필요한 만큼이나 어렵고 디버깅하기도 쉽지 않으므로 개념에 대해서 정확히 알고 있어야한다. Swift에서 작동하는 비동기 프로그래밍 방식에 대해 이야기하기 전에 먼저 알아야 할 배경지식들이 있다.

## Synchronous vs. Asynchronous

비동기가 무엇인지 알고 넘어가자! 그래야 아래에서 다룰 배경지식들이 왜 비동기를 이야기하는데 쓰이는지 더 감이 올 것이다. 구글에 "비동기 프로그래밍이란" 이라고 쳐봤더니 아래와 같이 나온다.

비동기 연산은 크기가 큰 파일을 읽거나 원격 컴퓨터에 접속하는 등 시간이 오래 걸리는 작업을 수행할 때 흔히 사용하는 방법이다. 먼저 메인 애플리케이션 스레드가 아닌 다른 스레드 혹은 OS에 의해 수행되기도 하기 때문에 비동기 연산을 요청한 후 결과가 나오기를 기다리는 대신 다른 작업을 할 수 있다.

이 두 문장만 읽어도 비동기 프로그래밍 개념을 이해하기 위해서는 `스레드`에 대해 공부해야한다는 것을 알 수 있다. 단순히 스레드가 무엇인지 아는 것에 더하여 멀티스레딩에 대해서 알아야 "메인 애플리케이션 스레드에서 하는 작업"과 "비동기로 요청된 오래걸리는 작업"이 어떻게 서로 방해하지 않고 `동시`에 작동하는지 이해할 수 있을 것이다.

컴퓨터 프로그래밍 언어는 기본적으로 왼쪽에서 오른쪽으로, 위에서 아래로 실행된다. 이 때 실행해야 할 task들이 A, B, C라면 A가 완료된 후에야 B를, B가 완료된 후에 C를 실행한다는 뜻이다. 이것이 흔히 사용되는 동기적 방식이다. (사실 흔히 사용된다기보다는 그냥 default다)

이런 경우에 네트워크에 접속해서 데이터를 받아오거나 크기가 큰 파일을 읽는 등 **시간이 오래걸리는 작업**이 중간에 껴 있다면 어떨까? A를 수행하는데 1초, B를 수행하는데 10초, C를 수행하는데 1초가걸린다고 해보자. 동기적 프로그래밍 방식으로 코드를 짠다면 C는 반드시 11초를 기다리고나서야 실행 될 것이다. 그렇다면 B를 서버에서 프로필 사진을 다운로드 받는 task, C는 받아온 이미지를 UI에 나타내는 task라고 해보자. 그러면 이미지를 다운받는 동안 휴대폰 화면을 업데이트 할 수 없으므로 frozen 상태가 되어버릴 것이고 이런 behavior는 사용성을 해친다.

이런 맥락에서 비동기 프로그래밍의 필요가 대두된다. 이미지를 아직 다운로드 받지 못했더라도 서버에 요청을 하고 응답을 받는 동안 UI는 계속해서 업데이트 시켜 줄 수 있어야하기 때문이다. (기본 이미지를 보여준다든지, 캐싱된 이미지를 보여준다든지 등) 아래 코드를 각각 동기/비동기적으로 실행시켜보자.

```swift
let taskA = {
    print("task A is Done.")
}

let taskB = {
    var i: Int = 0
    while i < 10 {
        sleep(1)
        i += 1
        print(".", terminator: " ")
    }
    print("task B is Done.")
}

let taskC = {
    print("task C is Done.")
}
```

```swift
// sync
taskA()
taskB()
taskC()
```

```swift
// async
taskA()
DispatchQueue.global().async { taskB() }
taskC()
```

![1](https://user-images.githubusercontent.com/35067611/104597167-55e56400-56b8-11eb-8027-a9599ad35479.png)

![2](https://user-images.githubusercontent.com/35067611/104597170-57169100-56b8-11eb-8ae3-4c86976c1e31.png)

결과는 10초를 기다리고 C가 실행되느냐, B가 실행되는 동안 코드의 실행흐름이 막히지 않고 (B가 완료될 때까지 기다리지 않고) C가 실행되느냐로 명확히 차이가 난다.

이제 동기, 비동기가 뭔지 알았으니 배경지식을 쌓고 Swift에서 비동기 프로그래밍을 구현하는 OperationQueud와 GCD 방식에 대해 알아보자.

## Process vs. Thread

스레드를 알려면 프로세스도 알아야한다. OS 수업에서 몇 주를 수업하고도 남을 주제이지만 간단하게 개념만 훑어보자.

- 프로세스는 하나의 프로그램이 메모리 상에서 `실행되는 단위`다.
- 스레드는 **프로세스 내에서 실행되는** `작업흐름의 단위`다.

보통 프로세스의 시작과 동시에 동작하는 스레드가 메인스레드가 되고 추가로 생성되는 스레드들을 서브 스레드라고 한다. 추가로 생성!? 그렇다. 프로세스는 둘 이상의 스레드를 가질 수 있도, 동시에 실행할 수도 있다. 이것이 멀티스레딩이다. 여기서 위에서 잠깐 스치듯 지나갔던 `동시`라는 단어가 다시 나온다. 사실 컴퓨터 입장에서 동시의 개념과 우리가 일상생활에서 사용하는 동시라는 단어의 개념에 갭이 좀 있다. 이것을 이해하기 위해 동시성과 병렬성에 대해 알아보자.

## Concurrency vs. Parallelism

사실 컴퓨터는 한번에 하나의 일만 수행한다 (싱글코어 기준!) 우리 눈에 "동시"에 여러 프로그램이 실행되는 것 처럼 보이는 것이지 실제로는 한번에 하나의 일을 매우 빠른 속도로 스레드 간 context switching을 하면서 수행한다. 이런 방식은 멀티스레딩을 통해 구현되며 `동시성(Concurrency)` 프로그래밍이라고 한다.

병렬성 프로그래밍은 "진짜로 한번에 여러 작업을 수행하는" 방식이다. 병렬적으로 프로그램들이 동시에 실행되기 위해서는 멀티스레딩만으로는 부족하다. 물리적으로 지원이 (멀티코어) 있어야 구현 가능하며 `병렬성(Parallelism)` 프로그래밍이라고 한다. 아래 그림을 보면 이해가 될 것이다.

<img width="640" alt="3" src="https://user-images.githubusercontent.com/35067611/104597178-58e05480-56b8-11eb-8c93-2c83bf843ad0.png">

- 동시성 : 통장을 만들러 온 N개의 대기열과 한 명 이상의 은행 직원
- 병렬성 : 통장을 만들러 온 N개의 대기열과 **N명의 은행직원**

또 다른 사진으로 이해해보자면 아래와 같다.

![4](https://user-images.githubusercontent.com/35067611/104597186-5b42ae80-56b8-11eb-842b-b4f2dda3d5b4.png)

자! 이제 프로세스와 스레드, 동시성과 병렬성에 대해서 간단히 알아보았으니 Swift에서는 비동기 프로그래밍을 어떻게 지원하는지 이해하기 수월해졌다.

## OperationQueue & Grand Central Dispatch

Swift에서 비동기 프로그래밍을 지원하는 도구에는 OperationQueue, GCD(Grand Central Dispatch) 두 가지가 있다. OperationQueue는 사실 GCD API 위에서 돌아간다. GCD는 더 로우레벨인 C 수준의 API로 더 빠르고 시스템과 직접적으로 연결되어있지만 OperationQueue는 Objective-C 수준의 API로 약간의 오버헤드를 동반한다. 결국 이 두 방식은 서로 대조되는 것이 아니라 기능적 차이가 있을 뿐 동일하게 Swift에서 비동기 프로그래밍을 가능하게 해주는 API인 것이다.

먼저 OperationQueue에 대해 알아보자! 공식문서를 보면 아래와 같이 정의되어있다.

A queue that regulates the execution of operations.

수행해야 할 task를 `operation` 단위로 묶어서 queue에 넣고 관리하는 것 같다. Operation이 한번 큐에 들어가면 반드시 수행이 끝나야 큐에서 나올 수 있다. 다시말해 한번 큐에 add 했다면 임의로 제거할 수 없다는 뜻이다. 또한 큐도 내부에 들어있는 모든 operation이 실행완료될 때까지 존재한다. 아직 실행이 끝나지 않았는데 OperationQueue를 제거하면 memory leak이 발생하게된다.

OperationQueue에 들어온 operation들은 readiness, priority level, and interoperation dependencies 등의 기준에 따라 순차적으로 실행된다. 이 기준들이 모두 동일한 우선순위를 갖는다면 들어온 순서대로 실행될 것이다. OperationQueue의 장점은 이런 우선순위에 대한 기준을 프로그래머가 정의해줄 수 있다는 것이다. 즉 비동기적으로 수행되어야 할 작업들에 대해서 순서를 지정할 수 있다는 장점이 있다 (dependencies).

또한 KVO(Key Value Observing)를 사용하여 큐 내부를 들여다볼 수 있다 (observable). 더하여 한번 실행시키면 큐에 대해 컨트롤을 할 수 없는 GCD와 달리 operation을 pause, cancel, resume 할 수 있다. GCD와 마찬가지로 OperationQueue도 thread safety가 보장되어 여러 스레드에서 sync를 맞추거나 추가적인 lock을 구현하지 않고 하나의 OperationQueue 객체를 사용해도 무방하다.

이번에는 GCD에 대해 알아보자.

An object that manages the execution of tasks serially or concurrently on your app's main thread or on a background thread.

GCD 역시 비동기 프로그래밍을 위해 메인 스레드가 아닌 background 스레드에서 여러 task들을 `dispatch queues`를 통해 serially/concurrently 실행시킨다. OperationQueue와 다른 점은 프로그래머가 컨트롤할 여지가 매우 적다는 것이다. 예를 들어 GCD의 경우, 큐에 작업이 들어오면 시스템이 알아서 thread pool에서 스레드를 골라 실행시켜주는데 이 때 어떤 스레드가 선택되는지 장담할 수 없다. (참고로 GCD도 thread-safe함)

GCD는 동기/비동기적으로 모두 실행시킬 수 있다. 동기적으로 실행시키면 모든 작업이 끝날 때까지 코드가 해당 라인에서 멈춰있을 것이고 비동기적으로 실행시키면 코드 실행흐름이 막히지 않고 계속 진행된다.

GCD의 실행은 serially/concurrently, synchronously/asynchronously 두 기준에 대해서 선택이 가능하다. 먼저 전자의 기준에 대해 알아보기 위해 serial queue, concurrent queue에 대해 그림으로 알아보자.

![5](https://user-images.githubusercontent.com/35067611/104597187-5b42ae80-56b8-11eb-8cc1-597fa7513433.png)

![6](https://user-images.githubusercontent.com/35067611/104597189-5bdb4500-56b8-11eb-8fb5-aea149458808.png)

- `Serial queue`에서는 한번에 하나의 작업이 실행된다. 실행 시작시간은 GCD가 알아서 컨트롤한다.
- `Concurrent queue`에서는 여러 작업이 한번에 실행된다. 함께 실행되지만 실행되는 순서는 큐에 들어온 순서이며 각 작업이 끝나는 시간이나 순서는 알 수 없다. 여러 작업을 동시에 실행시킬 때 다른 코어에서 실행시키거나 context switching을 하는 등의 의사결정 역시 GCD가 모두 알아서 한다.

GCD에는 세 가지 타입의 큐가 있다. (위 사진의 두 가지 + custom queue)

1) Main queue : 메인 스레드에서 실행되며 serial 큐이다.

2) Global queues : concurrent 큐이며 전체 시스템이 공유한다.

3) Custom queues : 프로그래머가 커스텀 할 수 있는 큐로 serial/concurrent 모두 구현할 수 있다.

GCD에서는 큐의 타입(serial vs. concurrent) 뿐 아니라 작동방식(synchronous vs. asynchronous)도 정의해줄 수 있다. 이것은 위에서 언급했던 동기/비동기 방식의 정의와 동일하다. 동기 방식의 GCD는 작업이 모두 끝날 때까지 다음 코드를 진행하지 않고 기다리는 반면 비동기 방식으로 GCD를 실행할 경우 현재 스레드를 block 하지 않고 다음 코드로 진행된다.

## 세 줄 요약

- OperationQueue는 작업 간의 dependency management와 같은 세부 컨트롤 요소가 필요할 때 사용한다.
- GCD는 단순히 코드블럭 단위의 작업을 serial/concurrent 큐에 넣어야 할 때 사용한다.
- OperationQueue와 GCD는 서로 반대되거나 대척점에 있는 개념이 아니라 프로그래밍을 할 때 필요에 따라 적절히 섞어서 사용돼야한다. (반드시 둘 중 하나만 사용해야하는 게 아니다!)

## References

[Grand Central Dispatch Tutorial for Swift 4: Part 1/2](https://www.raywenderlich.com/5370-grand-central-dispatch-tutorial-for-swift-4-part-1-2)

[제드님 블로그 - GCD - Dispatch Queue사용법 (1)](https://zeddios.tistory.com/516)

[[iOS Boostcourse] 동시성과 병렬성 그리고 비동기 프로그래밍](https://baked-corn.tistory.com/134)

[[iOS] 동시성, 비동기 프로그래밍 톺아보기](https://k-elon.tistory.com/20)

[Apple Developer Documentation - OperationQueue](https://developer.apple.com/documentation/foundation/operationqueue)

[Apple Developer Documentation - DispatchQueue](https://developer.apple.com/documentation/dispatch/dispatchqueue)
