---
layout: post
title: "iOS) setNeedsDisplay"
tags: [iOS, Swift]
comments: true
---

> 업데이트 싸이클 알아보기  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

지난 포스팅에서 layoutSubviews, setNeedsLayout, layoutIfNeeded에 대해서 공부했다. 이번에는 비슷한 결이지만 레이아웃이 아닌 디스플레이에 대해서 공부해보자!

## Display

먼저 디스플레이란 무엇인가? 디스플레이는 뷰와 하위뷰에서 sizing, positioning을 동반하지 않는 요소들을 포함한다. (위치와 크기는 레이아웃!) 이는 색상, 텍스트, 이미지, 코어 그래픽스 등이다. 디스플레이는 레이아웃과 마찬가지로 뷰 갱신을 하는 시스템 호출 메소드, 그 메소드를 트리거하는 메소드가 제공된다.

## draw(_: )

UIView는 sizing, positioning을 하는 layoutSubviews 메소드처럼 draw 메소드를 통해서 뷰의 컨텐츠를 그린다. 차이점이 있다면 draw 메소드는 하위뷰에 대해 연쇄적으로 호출되지 않는다. 하지만 layoutSubviews 메소드와 마찬가지로 draw 메소드 역시 시스템이 호출해주는 함수이지 우리가 명시적으로 호출하는 함수가 아니며 이를 트리거해주는 메소드가 있다.

## setNeedsDisplay()

이 메소드는 레이아웃의 setNeedsLayout과 동일하게 작동한다. 뷰의 컨텐츠 갱신을 예약하고 반환되며 다음 업데이트 싸이클에서 시스템은 모든 뷰들에 대해 이러한 갱신요청이 마크되었다면 draw를 호출해서 업데이트한다. 만약 뷰의 특정 부분을 다시 그리고 싶다면 (다음 업데이트 싸이클에) setNeedsDisplay 메소드를 호출하고 업데이트가 필요한 뷰를 넘겨주면 된다.

이 메소드를 명시적으로 호출하지 않아도 시스템은 UI components에 변경이 일어나면 이를 "dirty"로 체크하고 해당 뷰에 content updated flag를 세팅하여 다음 업데이트 싸이클에서 redraw 될 수 있도록 한다. 하지만 UI components와 직접적인 연관이 없는 요소가 변경되었고 이 변경이 뷰 갱신을 필요로 한다면 didSet property observer를 통해 적절하게 draw 메소드를 트리거할 수 있다. 아래의 예시를 보자.

```swift
class MyView: UIView {
    var numberOfPoints = 0 {
        didSet {
            setNeedsDisplay()
        }
    }

    override func draw(_ rect: CGRect) {
        switch numberOfPoints {
        case 0:
            return
        case 1:
            drawPoint(rect)
        case 2:
            drawLine(rect)
        case 3:
            drawTriangle(rect)
        case 4:
            drawRectangle(rect)
        case 5:
            drawPentagon(rect)
        default:
            drawEllipse(rect)
        }
    }
}
```

위 코드를 보면 numberOfPoints에 따라서 점, 선, 삼각형 등 화면에 그려지는 것들이 달라진다. numberOfPoints의 값이 바뀌면 화면에 그려지는 요소들이 달라져야 하므로 draw 함수를 재정의하되 직접 호출하는 것이 아니라 setNeedsDisplay를 통해 호출되도록 구현되어있다.

레이아웃에 layoutIfNeeded 메소드가 있는 것과 달리 디스플레이에는 즉시 컨텐츠 업데이트를 요청하는 메소드가 없다. 그 이유는.. "It is generally enough to wait until the next update cycle for redrawing views." 라고한다(?)

## 세 줄 요약

- 레이아웃과 마찬가지로 디스플레이에도 뷰의 컨텐츠를 갱신하는 메소드와 이를 트리거하는 메소드가 제공된다.
- 단, 디스플레이에서 layoutIfNeeded와 같은 즉시 갱신을 하는 메소드는 없다.
- UI Components와 관련 없는 요소에 의해 redraw 돼야 하는 상황이라면 didSet property observer, 그리고 override를 통해 재정의된 draw 함수를 사용하자!

## References

[Demystifying iOS Layout](https://tech.gc.com/demystifying-ios-layout/)
