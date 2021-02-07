---
layout: post
title: "iOS) setNeedsUpdateConstraints vs. updateConstraintsIfNeeded"
tags: [iOS, Swift]
comments: true
---

> 업데이트 싸이클 알아보기  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

이전의 두 포스팅(레이아웃, 디스플레이)에서 뷰의 갱신이 어떻게 이루어지는지 알아보았다. 마지막으로 Constraints(제약사항)에 대해서 공부해보자.

Auto Layout에서 뷰가 레이아웃되고 다시 그려지는 과정은 세 단계로 나뉜다. 먼저 뷰에 필요한 모든 제약들을 세팅하는 updating constraints. 그리고 레이아웃 엔진이 뷰와 모든 하위뷰의 프레임을 계산해서 재배치하는 layout pass. 마지막으로 뷰의 컨텐츠를 그리는 display pass로 이 싸이클이 완성된다. 이 중 가장 첫 단계에 대한 포스팅이다.

## updateConstraints()

이 메소드는 오토 레이아웃을 사용하는 뷰의 제약들을 동적으로 바꿀 수 있도록 하는 메소드다. 앞의 두 포스팅을 읽고왔다면 (꼭 먼저 읽기를 추천한다) 아래 한 문장으로 감을 잡을 수 있다.

Like `layoutSubviews` for layout and `draw` for content, `updateConstraints` should only be overridden and never explicitly called in your code.

이 부분만 읽어도 updateConstraints 메소드는 다음 업데이트 싸이클에서 제약사항들을 갱신하는 메소드이며 이를 트리거 할 수 있는 메소드가 제공되겠군! 이라고 짐작할 수 있고 실제로 그렇다. 이 메소드를 시스템에서 트리거하도록 internal flag를 세팅하는 상황들은 아래와 같다.

- activating or deactivating constraints
- changing a constraint’s priority or constant value
- removing a view from the view hierarchy

## setNeedsUpdateConstraints()

This method works similarly to `setNeedsDisplay` and `setNeedsLayout`.

이제 느낌이 온다! 다음 업데이트 싸이클에서 제약사항들에 대한 갱신이 일어나도록 updateConstraints를 예약하는 메소드이다.

## updateConstraintsIfNeeded()

This method is the equivalent of `layoutIfNeeded`, but for views that use Auto Layout.

오토 레이아웃을 사용하는 뷰에 대해서 layoutIfNeeded 메소드처럼 즉시 updateConstraints 메소드 호출을 시스템에게 요청하고 런 루프가 끝날때까지 기다리지 않는 메소드!

## invalidateIntrinsicContentSize()

이 메소드는 조금 생김새가 다르다! 오토 레이아웃을 사용하는 몇몇 뷰 중에 `intrinsicContentSize`(natural size of the view given its contents) 속성을 갖는 뷰들이 있다. 보통 intrinsicContentSize는 해당 뷰가 contain 하는 요소에 대한 제약에 의해 결정되지만 재정의 될 수도 있다. invalidateIntrinsicContentSize 메소드는 이런 intrinsicContentSize를 **다음 layout pass**에서 recalculate 되도록 flag를 표시한다. (즉, 현재 intrinsicContentSize가 낡았으니 새로 계산하라고 표시해주는 것과 같다)

## 세 줄 요약

- 레이아웃, 디스플레이와 마찬가지로 뷰 갱신을 위한 updateConstraints 메소드가 있으며 명시적으로 호출할 수 없지만 이를 트리거할 수 있는 메소드가 제공된다.
- 레이아웃, 디스플레이와 다르게 추가된 것이 있다면 intrinsicContentSize를 갖는 뷰에 대해서는 다음 layout pass로 넘겨서 recalculate를 하도록 표시하는 메소드가 있다.
- 레이아웃, 디스플레이, 제약사항은 모두 긴밀하게 연결되어있고 하나를 이해하면 전부를 이해할 수 있다!

## References

[Demystifying iOS Layout](https://tech.gc.com/demystifying-ios-layout/)
