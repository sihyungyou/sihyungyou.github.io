---
layout: post
title: "iOS) GCD Queue에 대하여"
tags: [iOS, Swift]
comments: true
---

> GCD Queue  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

## 동시성 프로그래밍 간단 정리

먼저 동시성 프로그래밍은 무엇이며 왜 필요할까? 모바일 환경에서는 UI에 View도 그려야하고 단순 작업도 해야하고 네트워크를 통해 이미지를 다운받는 등의 시간이 오래걸리는 일도 해야한다. 그런데 이 모든 것들을 메인 스레드에서 처리한다면 (별도의 처리가 없었다면 지금까지 그래왔을) 아무리 CPU가 빠르다고 한들 과부화되어 UI가 버벅거리는 등 사용성을 해치게 된다.

이런 문제를 해결하기 위해서 CPU는 메인 스레드에 몰린 작업들을 여러 스레드들에게 나누어주는데 이것이 동시성 프로그래밍의 의미이고 필요한 이유이다. 동시성과 비동기는 다른 개념인데 일단 동시성 자체에만 집중해서 이 배경상황을 이해해보자.

그렇다면 어떤 작업을 어떤 스레드에 어떤 기준으로 나눠줄 것인가? 이에 대해서는 걱정하지 않아도 된다 (iOS에서는..!) 우리는 단순히 작업들을 큐에 넣어주기만 하면 OS가 알아서, 정확히는 애플이 알아서 해준다. 즉, 우리가 할 일은 "작업이 너무 많다면 큐로 보내는 것" 뿐이다.

여기서 "알아서" 해주는 좋은 친구가 바로 `GCD`(Grand Central Dispatch), 그리고 GCD에서 사용하는, 우리가 작업을 넣는 큐가 `Dispatch Queue`이다. 즉, Dispatch Queue에 작업을 추가하면 GCD는 작업에 맞는 스레드를 자동으로 생성해서 실행하고, 작업이 종료되면 스레드를 제거한다. 워딩을 다시 해보자면 우리가 할 일은 "작업이 너무 많다면 Dispatch Queue로 보내는 것" 뿐이다.

정리하자면 **GCD는 멀티코어 환경에서 최적화된 프로그래밍을 지원하도록 애플이 개발한 기술**이고 **Queue이기 때문에 기본적으로 FIFO 방식**을 따른다!

## 그렇다면 비동기는?

그렇다면 그 중요하다는 비동기 개념은 어떤 시점에 들어오는 것일까. 동시성 프로그래밍이 수행됨으로써 메인스레드가 자신에게 할당된 너무 많은 작업들 중 하나를 다른 스레드에게 나눠주고자 Dispatch Queue에 밀어넣었다고 생각해보자. 이 때 메인스레드가 취할 수 있는 행동은 두 가지이다.

- 방금 밀어넣은 작업이 다른 스레드에서 끝나기를 `기다리고`, 기다린 후 메인 스레드에 밀려있던 작업을 시작한다.
- 방금 밀어넣은 작업은 다른 스레드가 알아서 해줄테니 신경끄고 메인 스레드의 다른 작업을 `바로` 시작한다.

아주 직관적이게도 1번이 `동기`, 2번이 `비동기` 방식이다. 여기서 한 가지 의문이 들 수 있는데 "기껏 큐에 작업 밀어넣어서 다른 스레드한테 일거리를 나눠주고는 그걸 기다리고 있으면 메인 스레드에서 처리하는 거랑 뭐가 다른가?" 그렇다. 다르지 않다. **동기적으로 큐에 넣는 코드를 짜면 실질적으로는 결국 메인 스레드에서 처리한다고 한다.**

## Why GCD?

이렇게 비동기, 동시성 프로그래밍을 하려면 결국 개발자는 스레드를 관리해야한다. 하지만 어떤 작업을 어떤 스레드에 어떻게 잘 나눠줄지 개발자가 직접 고민하고 코드를 짜는 것은 무지막지하게 어렵다. 그래서 애플은 이런 스레들 관리는 OS에게 맡기고 개발자는 작업이라는 한 단위에 집중해서 작업을 구성하고 그것을 큐에 넣기만 하면 되도록 GCD를 제공하는 것이다.

GCD를 사용하면 여러 측면에서 장점이 있다.

- 선언하기 편한 클로저, 프로듀서/컨슈머 관계를 표현하는 큐를 통해 더 쉽고 나은 표현으로 비동기 프로그래밍을 할 수 있다.
- 코드가 CPU 사이클을 더 효과적으로 활용하고 스레드를 재사용함으로써 효율성을 높인다.
- 서브 시스템에 독립적으로 OS가 관리하여 시스템 수준의 관리가 가능하다.
- Thread-safety를 보장하기 더 쉽다.

## GCD Queue 종류와 동작방식

![1](https://user-images.githubusercontent.com/35067611/104591878-afe22b80-56b0-11eb-945a-ad4346659452.png)

그럼 이제 (드디어) GCD가 사용하는 Dispatch Queue에 대해 알아보자. Dispatch Queue에는 위 그림과 같이 세 가지 큐가 있다. 사실 정확히는 두 가지 특성의 큐가 있고 세 가지 종류의 Dispatch Queue가 있다.

큐의 종류는 잠시 후에 보기로 하고 먼저 두 큐의 특성에 대해 알아보자. 잠시 비동기는 잊어두고 동시성 개념에 대해서만 생각해보면 Serial과 Concurrent는 결국 메인 스레드에서 분산 처리 시킨 작업을 각각 "다른 한 개의 스레드", "다른 여러개의 스레드"에서 처리하는 큐다. 이렇게 문자 그대로만 보면 무조건 Concurrent 큐가 어떤 상황에서도 좋을 것 같지만 이 둘의 쓰임새를 정확히 알고나면 Serial 큐가 필요할 때도 있음을 깨닫게된다.

- Serial Queue
![2](https://user-images.githubusercontent.com/35067611/104591886-b2dd1c00-56b0-11eb-8a75-5556696ed855.png)

Serial 큐의 특징은 큐에 담긴 작업들이 오직 하나의 스레드에만 분배된다는 것이다. 모든 작업들이 그 전 작업이 끝나길 기다렸다가 하나씩 실행 되기 때문에 task의 시작과 종료에 대한 순서 예측이 가능하다.

- Concurrent Queue
![3](https://user-images.githubusercontent.com/35067611/104591890-b375b280-56b0-11eb-881c-45b72d5467dc.png)

반면 Concurrent 큐에 담긴 작업들은 여러개의 스레드로 분배된다. 선입선출이라는 Queue의 특성상 작업들이 순서대로 분배되어 실행되긴 하겠지만, 그것들이 끝나는 순서는 알 수 없다. 이 때는 순서가 중요한 것이 아니라 많은 스레드를 활용해서 빨리 일처리를 하는것이 중요한 것!

두 큐를 함께 생각해보면 결국 `작업 순서의 중요도`에 따라 어떤 큐를 사용할지 정할 수 있음을 알 수 있다. 여러 작업을 처리하는데 있어서 순서는 중요하지 않고 최대한 빠르게 작업을 처리해야 한다면 `Concurrent` 큐를, 반대로 race condition 등을 고려해야하는 작업들 간의 순서가 중요한 상황이라면 `Serial` 큐를 사용하면 된다.

## GCD Queue 메서드 종류와 동작방식

잠시 중간점검을 해보자. 우리는 지금까지 총 네 가지 개념을 다루었다 (Concurrent, Serial, Async, Sync) 여기서 주의해야할 것이 Concurrent 큐는 동시에 여러 스레드가 도니까 이게 비동기(Async) 방식인가?라고 생각하게되는 것이다. 흔히 하는 착각이 Concurrent == Async, Serial == Sync 라는 일종의 공식인데 이는 잘못된 사실이다. 애초에 동시성과 비동기라는 서로 다른 개념이기 때문에 GCD를 활용할 때 아래와 같이 총 네 가지 조합이 나올 수 있다.

`SerialQueue.sync` : Serial 이니까 작업을 하나의 스레드로 보내고, Sync 이므로 동기방식으로 처리한다! ➡️ 메인 스레드는 자신이 다른 스레드로 분배한 작업이 끝날 때까지 **멈춰있고**(sync) 넘겨진 작업은 큐에 먼저 **담겨있던 작업들과 같은 스레드로** 보내지므로 해당 작업들이 모두 끝나야 실행된다(Serial Queue)

`ConcurrentQueue.sync` : Concurrent 큐니까 여러 다른 스레드로 작업들을 보내고, Sync 이므로 동기방식으로 처리한다! ➡️ 메인 스레드는 다른 스레드로 분배한 작업이 끝날 때까지 **멈춰있고**(sync), 넘겨진 작업은 큐에 먼저 담겨있던 작업들과 **다른 스레드로 보내질 수 있으므로** 해당 작업들이 모두 끝나지 않아도 실행된다(Concurrent Queue)

`SerialQueue.async` : Serial 큐니까 작업을 하나의 스레드로 보내고, Async이므로 비동기방식으로 처리한다! ➡️ 메인 스레드는 자신이 **다른 스레드로 작업을 분배하자마자 반환되어 다른 일을 하고**(async), 넘겨진 작업은 큐에 먼저 **담겨있던 작업들과 같은 스레드로 보내지므로** 해당 작업들이 모두 끝나야 실행된다(Serial Queue)

`ConcurrentQueue.async` : Concurrent 큐니까 작업을 여러 다른 스레드로 보내고, Async이므로 비동기 방식으로 처리한다! ➡️ 메인스레드는 **다른 스레드로 작업을 분배하자마자 반환되어 다른 일을 하고**(async), 넘겨진 작업은 큐에 먼저 담겨있던 작업들과 **다른 스레드로 보내질 수 있으므로** 해당 작업들이 모두 끝나지 않아도 실행된다(Concurrent Queue)

## Main, Global, Custom Queue

그럼 다시 큐의 "종류"로 돌아와보자.

먼저 Main Queue는 오직 한개만 존재하는 Serial 특성을 가진 큐다. 이곳에 할당된 작업은 메인 스레드에서 처리한다. 코드에서 별도의 처리를 해주지 않는 이상 이 Main Queue로 작업이 할당된다. 생각해보면 메인 큐는 메인 스레드로 작업을 보내고, 메인 스레드는 한개밖에 없으므로 메인 큐는 당연히 Serial 큐일 것이다.

두번째로 Global Queue는 Concurrent 특성을 가진 큐로 QoS에 따라 여섯가지 종류로 나뉜다. 작업을 큐로 보낼 때 순서가 상관없다면 Global Queue로 보내면 되며, 작업의 중요도는 QoS를 통해 결정해준다.

마지막으로 Custom Queue는 커스텀으로 직접 만드는 것이다. 디폴트로 Serial 특성을 가졌지만 Concurrent로 설정이 가능하며 당연히 QoS도 설정할 수 있다.

## GCD 사용 시 주의할 점

### 1. UI는 반드시 메인 스레드에서 처리한다

하나의 스레드(메인)에서 UI를 잘 처리하고 있는데 다른 스레드에서 끼어들면 UI가 버벅거리거나 의도치 않게 보일 수 있기 때문이다. 요지는 이미지 등을 `global` 에서 다운 받아오더라도, 해당 데이터를 `UIImageView`에 넣어서 **UI 업데이트 시켜주는 작업은** `main`에서 해야한다는 것!

### 2. 메인 스레드에서 다른 큐로 작업을 보낼 때는 sync를 사용해서는 안된다

위에서 설명했듯 메인 스레드는 기본적으로 UI 처리를 하는 최대한 방해받아서는 안되는 스레드이다. 그런데 sync 방식으로 다른 큐에 작업을 넣으면 결국 그 작업이 모두 처리되기를 기다리게 되므로 역시 UI 업데이트가 지연되므로 UI에 영향을 주지 않을 수 없다. 그러니 메인스레드에서는 항상 비동기 방식으로 작업을 보내야한다.

### 3. 현재와 같은 큐에 sync로 작업을 보내면 안된다

![4](https://user-images.githubusercontent.com/35067611/104591894-b40e4900-56b0-11eb-9d6e-4cd747b4a42d.png)

이는 위 사진과 같은 상황인데 이렇게 되면 데드락 상황이 발생한다. 천천히 이 상황을 살펴보자

1. Global Queue에 Task A가 들어왔고 스레드 2번에 할당되었다.
2. 스레드 2번이 Task A를 처리하려고 보니 Global Queue에 Task B를 동기방식으로 보내야한다. 보내주자.
3. 단, 동기방식이므로 스레드 2번은 Task B를 Global Queue에 밀어넣고 Task B가 완료될 때 까지 기다린다.
4. Global Queue는 Task B가 들어왔으므로 스레드에 할당해야하는데 이 때 스레드 2번에 할당했다.
5. 2번은 Task B를 밀어넣고 대기 상태에 들어갔는데 그 일을 Global Queue가 다시 자기자신에게 준 꼴이다.

참고로 큐에서 사용하고있는 쓰레드 객체는 정해져 있다. 예를 들어 디폴트 글로벌큐는 쓰레드 2, 3, 4번만 사용하고, 백그라운드 글로벌큐는 쓰레드 5, 6번만 사용하는 식으로. 그런데 같은 큐에 보내면 같은 스레드에 작업을 할당할 가능성이 있으므로 데드락이 발생할 수 있는 것이다. (물론 같은 큐이더라도 다른 스레드에 보내지면 데드락이 발생 안할 수는 있다.)

여기서 한 가지 방법이 있다면 Global Queue는 QoS에 따라 각각 다른 큐 객체를 생성한다. 즉 DispatchQueue.global(qos: .utility) 와 DispatchQueue.global()는 다른 큐로, 쓰레드가 겹칠일이 없기 때문에 데드락 발생 가능성이 없다.

### 4. 메인스레드에서 DispatchQueue.main.sync는 사용하면 안된다

메인 스레드에서 메인 큐로 동기방식으로 작업을 보낸다는게 무슨 의미일까? 일단 메인 큐는 Serial 큐이며 메인 스레드에만 작업을 할당한다. 그렇다면 이는 메인 스레드가 메인 스레드에게 동기방식으로 작업을 할당하는 꼴로, 방금 위에서 말한 데드락 상황을 자초하는 것과 다름이 없다.

## NSOperationQueue vs. GCD Queue

호우 힘들다 보류..

## References

[[iOS] 차근차근 시작하는 GCD - 5](https://sujinnaljin.medium.com/ios-차근차근-시작하는-gcd-5-c8e6eee3327b)
