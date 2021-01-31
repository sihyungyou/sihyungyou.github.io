---
layout: post
title: "iOS) setNeedsLayout vs. layoutIfNeeded"
tags: [iOS, Swift]
comments: true
---

> 업데이트 싸이클 알아보기  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

iOS 개발을 하다보면 심심치 않게 `setNeedsLayout`, `layoutIfNeeded` 등 중요한 것 같으면서도 왜 사용하는지 정확히 이해하지 못하고 사용하는 함수들을 만나게된다. 오늘은 이렇게 뷰의 업데이트 주기와 관련있는 함수들에 대해서 공부해보려한다.

## Main Run Loop

먼저 iOS 애플리케이션이 어떻게 작동하는지부터 알아보자.

![1](https://user-images.githubusercontent.com/35067611/104595160-8b3c8280-56b5-11eb-9e06-ee2afae77021.png)

위 사진처럼 iOS 애플리케이션은 유저 인터랙션을 모두 이벤트 큐에 넣고, 런 루프가 큐에서 이벤트를 하나씩 빼와 애플리케이션 오브젝트에게 dispatch 해주면서 처리하는 방식으로 동작한다. 이 이벤트 핸들러들이 모두 처리되고 반환되면 다시 메인 런 루프로 돌아오는 것이다. 그리고 이 때 핸들러가 처리되면서 뷰에 일어난 변경사항을 적용하여 뷰들을 다시 그리고 배치하는 작업을  `update cycle`가 담당하게 된다.

## Update Cycle

업데이트 싸이클은 애플리케이션의 컨트롤이 이벤트 핸들링 코드를 모두 처리하고 메인 런 루프로 돌아오는 "시점"을 의미한다. 이 시점에 시스템은 layout, display, constraint를 갱신한다.

![2](https://user-images.githubusercontent.com/35067611/104595171-8e377300-56b5-11eb-8f6b-f5de55c82b34.png)

만약 어떤 뷰에 이벤트 핸들러를 통해 수정사항을 요청한다면 시스템은 즉시 갱신하는 것이 아니라 이 업데이트 싸이클이라는 시점에 왔을 때 갱신한다. (정확히는 "다음 업데이트 싸이클에") 여기서 알 수 있는 점은 유저 인터랙션이 일어나는 시점과 유저 인터랙션을 통해 발생하는 뷰의 수정사항이 실제로 갱신되는 시점에 시간차가 존재한다는 것이다. 물론 이 런 루프가 refresh 되는 데에는 1/60초 밖에 걸리지 않기 때문에 유저는 이러한 시간차를 인지하지 못하고 애플리케이션을 사용하게된다. 그렇다면 이런 시간차에 대해서 알아야 할 이유가 뭘까? 이러한 흐름을 이해하지 못하면 개발자가 원하는대로 정확히 뷰가 업데이트 되도록 프로그램을 짤 수 없다. 항상 다음 업데이트 싸이클을 기다리는 것이 아니라 즉시 업데이트를 하도록 해야 하는 경우도 있고, 업데이트 하는 과정을 재정의하여 어떤 action을 실행시킬 수도 있기 때문이다.

## Layout

먼저 레이아웃에 대해 알아보자. 레이아웃이란 스크린의 크기와 위치를 의미한다. (잘 알고 있겠지만 모든 뷰는 부모뷰의 좌표계에서 위치와 크기를 나타내는 frame 값을 갖는다) 뷰는 우리가 시스템에게 뷰의 레이아웃이 바뀌었다고 알릴 수 있는 메소드, 레이아웃이 갱신되었을 때 action을 취할 수 있도록 재정의 할 수 있는 메소드를 제공한다

### layoutSubviews()

이 메소드는 UIView의 메소드로 해당 뷰와 모든 하위뷰의 **repositioning**, **resizing**을 handle한다. 즉, 시스템은 뷰의 프레임을 다시 계산해야할 때 이 메소드를 호출한다. 그러므로 프레임을 세팅하거나 위치와 크기에 대해서 specify 해야 할 때는 이 메소드를 override 해야한다. 단, 해당뷰와 모든 하위뷰에 대해서 작동하기 때문에 이 메소드는 상당히 expensive하다. (모든 하위뷰의 layoutSubviews 메소드도 각각 불린다는 뜻이다) 그러므로 이 메소드는 개발자가 명시적으로 호출해서는 안되고, 시스템이 호출하도록 둬야한다. (그래서 항상 override를 통해 재정의만 한다)

다만 시스템으로 하여금 이 메소드가 호출될 수 있도록 트리거하는 less expensive한 메소드가 존재하는데 그것이 바로 setNeedsLayout, layoutIfNeeded 이다. 이 메소드들은 **업데이트 싸이클이 아니라 메인 런 루프 시점에** layoutSubviews를 트리거한다! 또한 이런 명시적인 메소드 외에도 시스템 내부에서 자동으로 layoutSubviews를 트리거하는 경우가 있다. 이들은 모두 레이아웃에 변경이 일어나는 이벤트들인데 아래와 같다.

- Resizing a view
- Adding a subview
- User scrolling a UIScrollView (layoutSubviews is called on the UIScrollView and its superview)
- User rotating their device
- Updating a view’s constraints

위 상황들은 시스템에게 뷰의 위치가 다시 계산되어야 한다고 알리고 layoutSubviews 메소드가 호출된다.

참고로 layoutSubviews 메소드가 완료되면 해당 뷰를 소유하는 뷰컨트롤러에서 `viewDidLayoutSubviews` 메소드가 트리거된다. layoutSubviews는 레이아웃의 업데이트 이후 불리는 유일한 메소드이기 때문에 이런 레이아웃과 sizing에 의존하는 로직은 `viewDidLoad`나 `viewDidAppear`가 아닌 viewDidLayoutSubviews에 정의해주어야한다.

layoutSubviews가 어떤 메소드이고 언제 불리는지에 대해서는 감을 잡았다. 그렇다면 이 메소드를 업데이트 명시적으로 호출할 수 있는 두 메소드에 대해서 알아보자

### setNeedsLayout()

layoutSubviews 메소드를 호출하는 가장 비용이 적게 드는 트리거다. 이 메소드는 시스템으로 하여금 자신을 호출한 뷰의 레이아웃이 다시 계산되어야 한다고 알려주고 즉시 반환된다. 대신, 뷰는 다음 업데이트 싸이클에 갱신된다. 즉 바로 다음 업데이트 싸이클에 레이아웃이 갱신될 수 있도록 마크해두는 것이다. (layoutSubviews를 예약하는 것)

고민 : 명시적으로 호출하고 `비동기적`으로 메소드는 반환되지만 여전히 유저 인터랙션과의 시간차가 발생하는데 어떤 이점이 있을까? (아래 글에서 마지막 부분 잘 와닿지 않는다)

Instead, the views will update on the next update cycle, when the system calls layoutSubviews on those views and triggers subsequent layoutSubviews calls on all their subviews. There should be no user impact from the delay because, even though there is an arbitrary time interval between when setNeedsLayout returns and when views are redrawn and laid out, it should never be long enough to cause any lag in the application.

### layoutIfNeeded()

setNeedsLayout과 마찬가지로 layoutSubviews를 트리거하는 또 다른 메소드이다. 명확한 차이점이 있는데 다음 업데이트 싸이클에 layoutSubviews를 예약하는 것이 아니라 시스템에게 즉시(immediately) 레이아웃 업데이트를 갱신해달라고 요청한다. 즉 다음 업데이트 싸이클까지 기다리지 않고 바로 layoutSubviews를 호출하는 메소드이다.

만약 한 뷰가 setNeedslayout을 호출하고 layoutIfNeeded를 호출한다면 어떻게 될까? 전자는 다음 업데이트 싸이클에 뷰 갱신을 예약했고, 후자는 즉시 뷰 갱신을 요청했다. 일단 layoutIfNeeded 때문에 그 즉시 뷰는 업데이트 될 것이다. 그렇다면 setNeedsLayout이 예약한 다음 업데이트 싸이클에서의 layoutSubviews는 호출될 필요가 없지 않을까? 실제로 예약된 뷰 갱신은 일어나지 않는다. 비슷한 예로 layoutIfNeeded를 연속으로 두번 호출한다고 해도 layoutSubviews 메소드는 한번만 호출된다.

그런데 어차피 다음 업데이트 싸이클까지 기다리는 시간차가 유저에게 영향을 미치지 않을정도로 짧은데 굳이 즉시 뷰 갱신을 하는 이유는 뭘까? 이 메소드는 레이아웃에 의존해야하거나 다음 업데이트 싸이클까지 기다릴 수 없을 때 유용하다. (이런 상황이 아니라면 setNeedsLayout을 호출하는 것이 옳다) 특히 애니메이션에서 자주 사용된다. 그 이유는 애니메이션이 시작되기 전에 모든 레이아웃이 업데이트 된 상황이어야하기 때문이다.

## 세 줄 요약

- layoutSubviews는 시스템에서 호출하는 뷰 갱신 메소드로 매 업데이트 싸이클에 변경사항이 있는지 확인하고 한꺼번에 갱신한다.
- layoutSubviews는 호출하는 뷰의 모든 하위뷰들도 이 메소드를 호출하기 때문에 매우 비용이 많이 듦으로 직접 호출하지 말고 명시적으로 호출해야한다면 setNeedsLayout, layoutIfNeeded를 사용해야한다.
- setNeedsLayout은 다음 업데이트 싸이클에 layoutSubviews를 예약하고 반환되는 비동기 메소드이고, layoutIfNeeded는 즉시 layoutSubviews 메소드를 호출하는 동기 메소드이다.
- (네줄 요약?) 다음에는 display, constraint에 대해 공부해서 포스팅해봐야겠다!

## References

[Demystifying iOS Layout](https://tech.gc.com/demystifying-ios-layout/)

[[ios] setNeedsLayout vs layoutIfNeeded](https://baked-corn.tistory.com/105)

[제드님 블로그 - View/레이아웃 업데이트 관련 메소드](https://zeddios.tistory.com/359)
