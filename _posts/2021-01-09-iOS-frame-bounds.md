---
layout: post
title: "iOS) Frame vs. Bounds"
tags: [iOS, Swift]
comments: true
---

> 프레임과 바운드의 차이점 알아보기  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

iOS 앱개발을 하다보면 View를 필수적으로 다루게되고, View를 갖고 놀다보면 frame, bounds라는 속성을 반드시 만나게된다. 둘의 명확한 차이를 잘 몰라도 웬만해서는 frame을 쓰든 bounds를 쓰든 정상적으로 좌표나 길이, 너비를 구하고 조정할 수 있었다. 그래도 둘이 나누어져있는데는 분명 이유가 있을테니 알아보도록 하자.

## Definition

애플 공식문서에 따르면 frame의 정의는 다음과 같다.

The frame rectangle, which describes the view’s location and size in its superview’s coordinate system.

여기서 주목할 점은 "in its **superview's coordinate system**" 즉, frame은 superview의 좌표계를 기준으로 해당 view가 위치하는 location을 나타낸다는 것이다. 참고로 이 때 superview라 함은 "한 단계" 상위뷰를 의미한다.

bounds도 마찬가지로 애플 공식문서를 보자.

The bounds rectangle, which describes the view’s location and size in its own coordinate system.

frame과 다른 점은 "in its **own coordinate system**" 부분으로, bounds는 자기자신의 좌표계를 기준으로 location이나 size를 표현하는 property임을 알 수 있다.

frame, bounds 모두 UIView의 instance property로 타입은 CGRect이다.

```swift
var frame: CGRect { get set }
var bounds: CGRect { get set }
```

(instance property, CGRect도 다시 공부해봐야지..)

## 차이점

이렇게 기준이 되는 좌표계가 다르다면 같은 위치에 있더라도 origin이 달라질 수 있음을 쉽게 떠올릴 수 있다. 예를 들어 아래와 같이 View가 놓여져 있고 하얀 View가 superview라고 하자. (왼쪽사진)

이 때 frame origin 좌표는 superview의 x, y축으로부터 떨어진 만큼 계산된다. 대강 (20, 60) 정도 될듯? 하지만 bounds origin 좌표는 항상 (0, 0)일 것이다. superview와의 관계와 상관없이 자기자신의 좌표계 기준 origin은 항상 (0, 0)이기 때문에 그렇다. size의 경우, frame과 bounds 모두 동일하다.

![1](https://user-images.githubusercontent.com/35067611/104597528-ca200780-56b8-11eb-911a-534b0350f258.png)

그렇다면 size는 항상 같고 origin만 달라지는 걸까? 고작 그 차이를 위해 frame과 bounds를 따로두었을까? frame의 size는 이렇게 View가 x축이나 y축에 평행할 때는 bounds와 항상 똑같지만 View를 회전시키면 이야기가 달라진다. 그 이유는 frame은 사실 단순히 View가 아니라 해당 View를 감쌀 수 있는 사각형 모양의 View이기 때문이다.

아니 이게 무슨말이냐!? 오른쪽 사진을 보면 View를 감싸는 빨간 사각형의 크기가 달라진 것을 알 수 있다. 이 때 bounds 값은 왼쪽 사진과 동일하다. 실제로 View의 크기나 좌표가 바뀐것은 전혀 없기 때문이다. 단, 회전되었을 뿐이다. bounds와는 달리 frame의 origin, size 값은 모두 더 커진 빨간 사각형 기준으로 계산된다. 다시말해 frame은 View가 회전하면 그에 맞춰 해당 View를 모두 감쌀 수 있도록 더 커진다!

단순히 기준이 되는 좌표계가 다른 것 외에도 이런 성질이 있었구나! 부캠 10주동안 전혀몰랐쥬..? 아무튼 이런 성질을 이용해서 frame은 애니메이션에 사용되기 매우 적합하다고한다.

frame은 그렇다 치고.. bounds 얘는 그럼 진짜 별거없네? 그럴리가 ㅠ bounds는 그렇게 만만한 친구가 아니었다. 아래 두 사진을 보자!

![2](https://user-images.githubusercontent.com/35067611/104597538-cc826180-56b8-11eb-81ea-2b8578d83126.png)

위 사진에서 blueView는 redView의 subView이다. 두 사진만 놓고 보면 코드상에서 뭔가 blueView의 좌표를 바꾸어준 것 처럼 보인다. 하지만 실제로 작성한 코드는 redView의 bounds를 바꾸는 코드였다.

```swift
redView.bounds.origin.x = 50
redView.bounds.origin.y = 60
```

왜 Why!? redView의 bounds를 건드렸는데 엉뚱하게 blueView가 왼쪽으로 50, 위쪽으로 60만큼 이동한걸까? bounds를 변경한다는 것은 해당 위치에서 View를 다시 그린다는 의미이다. 그런데 bounds를 바꾸어도 상위 view와는 아무 상관이 없으므로 흰색의 backgroundView 좌표계를 기준으로 한 redView의 위치에는 변함이 없다. 하지만 (50, 60)이라는 새로운 위치에서 redView를 그리게 되는 것이다.

이렇게 bounds의 origin을 변경함으로써 우리 눈에 보이는 View를 달리하는 것이 스마트폰에서 항상 사용하는 ScrollView의 원리이다. 아래는 제드님 블로그에서 가져온 GIF인데 이 원리를 잘 설명하고 있다.

![3](https://user-images.githubusercontent.com/35067611/104597541-cdb38e80-56b8-11eb-9979-55e91cc6f712.gif)

## 언제 무엇을 쓰는가?

제드님 블로그에 따르면...

- frame은 View의 위치나 크기를 설정할 때 사용 (Use this rectangle during layout operations to **set the size and position the view**.)
- bounds는 View 내부에 그림을 그릴 때, transformation 후 View의 크기를 알고싶을 때, 하위 View를 정렬하는 것과 같이 내부적으로 변경하는 경우에 사용

라고 한다. 왜 와이!? 일단 frame은 위치를 잡을 때 항상 쓰인다. 실제로 Storyboard에서 View의 위치를 지정해줄 때 쓰이는 x, y 좌표는 frame 값이다.

![4](https://user-images.githubusercontent.com/35067611/104597549-cee4bb80-56b8-11eb-8259-4c97f665bb0d.png)

bounds는 superview의 좌표시스템을 따르지 않고 자기자신만의 좌표시스템가 기준이기 때문에 위치정보가 없으며 View의 **크기만** 변경할 수 있다. 띠용? 위에서는 bounds.origin.x, y 값을 바꿨는데!? 이것은 실제로 View의 위치를 변경하는게 아니라 내부 View가 보여지는 화면(창문이라고 이해해보자)의 위치를 바꾸는 것이다.

## 세 줄 요약

- Frame은 superview의 좌표계기준, bounds는 자기자신의 좌표계를 기준으로 origin과 size가 결정된다.
- Frame은 단순히 UIView와 동일한 위치나 크기가 아니라 view가 회전되었을 때도 감쌀 수 있는 사각형을 의미한다.
- Bounds를 옮기면 마치 subview가 움직이는 것 처럼 보이는데 이 현상은 superview의 frame은 변하지 않고 그대로 있되, subview들을 그리는 좌표계의 기준이 달라졌기 때문이다.  

## References

[Apple Developer Documentation - Frame](https://developer.apple.com/documentation/uikit/uiview/1622621-frame)

[Apple Developer Documentation - Bounds](https://developer.apple.com/documentation/uikit/uiview/1622580-bounds)

[[ios] Bounds vs Frame?](https://baked-corn.tistory.com/81)

[Sean Allen 유튜브](https://www.youtube.com/watch?v=Nfzy1qgxSAg&feature=youtu.be)

[제드님 블로그 - Frame과 Bounds의 차이 (1/2)](https://zeddios.tistory.com/203)

[제드님 블로그 - Frame과 Bounds의 차이 (2/2)](https://zeddios.tistory.com/231)
