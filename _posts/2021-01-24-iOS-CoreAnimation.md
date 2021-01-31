---
layout: post
title: "iOS) Core Animation 맛보기"
tags: [iOS, Swift]
comments: true
---

> 말로만 듣던 CA에 대하여  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

# Core Animation API에 대해서

오늘은 애니메이션을 구현할 때 자주 사용되는 (사실 다른 것들도 엄청 많은데 부스트포켓 프로젝트 때 사용한) CALayer, CABasicAnimation, CAShapeLayer에 대해서 알아보자. 이들은 모두 Core Aminamtion API에서 제공되는 클래스들로 Core Animation 자체에 대해서도 간단히 배경지식을 쌓고 가려한다.

## Why Layer?

먼저 레이어에 대해 알아봄과 동시에 Core Animation이라는 API의 배경을 같이 생각해보겠다.

모든 뷰(View)는 레이어(layer)가 그 밑받침에 있다. 그렇다면 뷰란 무엇인가? 뷰는 이미지, 비디오, 글자들을 보여주는 객체이며 터치, 제스쳐 등의 유저가 행하는 것을 잡아서 처리할 수 있다. 그리고 이 뷰는 CALayer 객체인 `layer` 프로퍼티를 갖고 있으며 shadow, rounded corner, colored border, 3d transform, masking contents, animation 등의 기능을 제공한다.

지금까지 레이어에 직접적으로 접근하여 코딩을 하지 않았더라도 항상 iOS는 화면을 그릴 때 레이어를 사용하고 있었다. 실제로 UIView는 레이아웃과 터치 이벤트 처리 등 많은 작업을 한다. 이 중에서 뷰 위에 컨텐츠나 애니메이션을 그리는 행위는 직접 다루지 않고 UIKit가 CoreAnimation에게 위임한다. 즉, 실질적으로 뷰 위에 컨텐츠나 애니메이션을 그리는 행위는 CALayer에서 담당하게 된다.

결국 UIView는 CALayer를 감싸고 있는 것에 불과하다. 그렇기 때문에 UIView의 bounds가 변경되면 UIView는 자신의 루트 layer의 bounds를 변경한다. 이렇게 루트 레이어의 레이아웃은 속해있는 뷰에 맞추어 자동으로 변경된다. (서브 레이어는 자동으로 맞추어지지 않는다)

![1](https://user-images.githubusercontent.com/35067611/105620998-3089f000-5e46-11eb-806d-81c57b4f8759.png)

CALayer에 대해 알아보기 전에 먼저 애플이 앱에서 그래픽을 지원해온 경로를 살펴보자. 사용자가 느끼기에 부드럽고 빠른 애니메이션을 애플은 1초당 60프레임을 보여줌으로써 구현한다. 이러한 rate를 유지하기 위해 기본적이면서 강력한 GPU 위에서 바로 돌아가도록 한 레이어가 `OpenGL`이다.

OpenGL은 iOS 기기의 그래픽 하드웨어에 대한 가장 저수준의 API를 제공한다. 하지만 여기에는 tradeoff가 있는데 OpenGL은 뼈대만 있고 간단한 작업에도 많은 코드를 구현해야한다는 것이다. 이러한 점을 개선하기 위해서 `Core Graphics`가 개발된다. 이것은 OpenGL보다 살짝 높은 수준의 API로, 더 적은 코드로 많은 기능을 구현할 수 있게해준다. 하지만 기능 자체는 여전히 저수준이었기에 이것을 한번 더 단순화하기 위해 `Core Animation`이 개발된 것이다. 그리고 이게 바로 오늘의 주제인 `CALayer` 클래스를 제공하는 기초적인 저수준 그래픽 API이다.

![2](https://user-images.githubusercontent.com/35067611/105621000-341d7700-5e46-11eb-9efd-1205536b009e.png)

한마디로 Core Animation은 iOS / OS X에서 CPU에게 부담을 주지 않으면서 사용 할 수 있는 그래픽 렌더링 및 애니메이션 **인프라**이다. 여기서 주의해야할 점은 Core Animation이 **드로잉 시스템 자체가 아니라 드로잉 작업을 하드웨어로 전달한다는 것**이다. 그리고 그 핵심에 콘텐츠를 관리하고 조작하는 레이어 객체가 있다. 레이어는 콘텐츠를 그래픽 하드웨어에서 쉽게 조작될 수 있는 비트맵 형태로 캡쳐한다.

애플은 Core Animation의 advanced한 기능들이 일반적인 앱에서는 사용되지 않는다고 판단하여 `UIKit`를 만들었다. 그리고 이 UIKit이 iOS 그래픽에 있어서 가장 고수준의 접근을 제공한다. UIkit의 장점은 어떤 레벨의 그래픽 접근이든 선택할 수 있다는 것이다. 이는 정확히 어느 정도의 기능을 요구하는지에 따라 정확히 고를 수 있고 불필요한 코드를 작성하지 않아도 되도록 한다. (하지만 이런 장점에 비해 단점도 있는데 고수준의 그래픽 API일수록 더 적은 기능을 제공한다는 것이다)

CALayer가 있기에 iOS는 앱의 view hierarchy에 대한 비트맵 정보를 빠르고 쉽게 생성할 수 있다. 그리고 이 정보는 Core Graphics를 거쳐 OpenGL에 전달되어 GPU에 의해 렌더링 된 후 기기의 화면에 나온다. 대부분의 경우 이렇게 CALayer에 직접 사용하는 것이 필수는 아니지만 저수준의 API는 개발자들에게 **더 많은 기능과 유연한 customization을 제공하기 때문에** CALayer의 존재와 그 내부 속성에 대해서 알 필요가 있다.

## CALayer의 여러가지 속성

먼저 레이어 계층에 접근하는 것은 어렵지 않다. 위에서 말했듯 모든 뷰는 레이어가 밑받침이 된다고 했으니 `myView`가 있다고 할 때, 아래와 같이 layer에 접근하여 여러가지 property 값을 설정해줄 수 있다.

```swift
myView.layer
```

이 포스팅에서는 어떻게 각각의 property를 설정하고 어떤 의미를 갖는지보다 CALayer가 어떤 의미와 효용을 갖는지와 이 계층을 건드렸을 때 퍼포먼스적으로 어떤 효과가 있는지에 더 집중하겠다.

CALayer에 접근해서는 `cornerRadius`, `shadowColor`, `shadowOffset`, `borderColor`, `borderWidth`, `masksToBounds`, `backgroundColor` 등 여러가지 속성들에 대해 정의해줄 수 있다.

## CABasicAnimation

애플 공식문서에 따르면 CABasicAnimation은 "An object that provides basic, single-keyframe animation capabilities for a layer property", 즉 레이어 속성에 대해 애니메이션 기능을 제공하는 객체이다.

![3](https://user-images.githubusercontent.com/35067611/105621001-354ea400-5e46-11eb-9f5d-1e2f49d6e74a.png)

와 뭐가 많다.. 이걸 언제 다 공부하지? 오늘은 이 중에서 CABasicAnimation에 대해서만 알아보도록하자.. 공식문서의 Overview를 살펴보았다.

CABasicAnimation 인스턴스는 애니메이션 방식을 특정하는 key path를 설정하여 생성한다. 예를 들어 레이어의 scalar(single value), 그 중에서도 opacity 값을 0에서 1로 애니메이션 시킨다고 생각해보자.

```swift
let animation = CABasicAnimation(keyPath: "opacity")
animation.fromValue = 0
animation.toValue = 1
```

아래 코드는 backgroundColor를 변화시키는 코드이다.

```swift
let animation = CABasicAnimation(keyPath: "backgroundColor")
animation.fromValue = NSColor.red.cgColor
animation.toValue = NSColor.blue.cgColor
```

이어서 아래 코드는 non-scalar property인 position을 변화시키는 코드이다.

```swift
let animation = CABasicAnimation(keyPath: "position")
animation.fromValue = [0, 0]
animation.toValue = [100, 100]
```

위 세 가지 경우 모두 fromValue, toValue라는 값을 지정해주는 것을 알 수 있다. 직관적으로 뭔가 fromValue의 값에서 toValue의 값으로 animation이 동작한다는 것을 느낄 수 있다. 실제로 애플공식문서에서 설명하는 definition도 다음과 같다:

`fromValue` : Defines the value the receiver uses to **start** interpolation.

`toValue` : Defines the value the receiver uses to **end** interpolation.

`byValue` : Defines the value the receiver uses to **perform** relative interpolation.

이 세 property들은 모두 optional이며 적어도 두개는 채워져아 한다. (그런데 하나만 non-nil일 때와 모두 nil인 경우에 대해서도 써있다..?) 각각이 옵셔널일 때 animation의 behavior가 달라진다.

- fromValue, toValue 모두 non-nil : fromValue → toValue로 연결(interpolates)
- fromValue, byValue 모두 non-nil : fromValue → (fromValue + byValue)
- byValue, toValue 모두 non-nil : (to - byValue) → toValue
- fromValue만 non-nil : fromValue → current presentation value of the property
- toValue만 non-nil : current value of key path → toValue
- byValue만 non-nil : current value of key path → (current value + byValue)
- 모두 nil : previous value of key path → current value of key path

## CAShapeLayer

애플공식문서에 따르면 CAShapeLayer는 "A layer that draws a cubic Bezier spline in its coordinate space."라고 설명되어있다. Bezier spline이 뭐지..? 뭔가 도형을 그리는 것 같긴한데 일단 속성을 살펴보자.

CAShapeLayer에는 여러 속성(fillColor, lineWidth 등)이 있지만 그 중에서도 path라는 속성이 있다. 여기에 UIBezierPath를 선언하고 정의해서 넘겨주면 그 path를 따라 그대로 그리겠다는 뜻 같다.

```swift
let shapeLayer = CAShapeLayer()
shapeLayer.frame = CGRect(x: 0, y: 0,
                          width: width, height: height)

// 위에서 path라는 변수를 BezierPath 객체로 선언했다고 치고,
shapeLayer.path = path
shapeLayer.strokeColor = UIColor.red.cgColor
shapeLayer.fillColor = UIColor.yellow.cgColor
shapeLayer.fillRule = kCAFillRuleEvenOdd
```

![4](https://user-images.githubusercontent.com/35067611/105621002-35e73a80-5e46-11eb-9e7f-b867d6b073f4.png)

애플공식문서에서 보여주는 예제 코드의 일부와 결과인데 이렇게 화려한 걸 예제로 그릴 일이냐고..

아무튼 정리하면 CAShapeLayer는 CALayer의 서브클래스로 cubic Bezier spline(?)를 그리는 객체이다. 흠..일단 BezierPath만 사용해도 그릴 수 있는 것들과 설정할 수 있는 attribute들에 대해서 CAShapeLayer에서도 그릴 수 있는 것 정도로만 이해하고 있다.

## References

[애플공식문서 - About Core Animation](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40004514)

[애플공식문서 - CABasicAnimation](https://developer.apple.com/documentation/quartzcore/cabasicanimation#2776772)

[A Beginner's Guide to CALayer](https://www.appcoda.com/calayer-introduction/)

[What is the full Keypath list for CABasicAnimation?](https://stackoverflow.com/questions/44230796/what-is-the-full-keypath-list-for-cabasicanimation)

[애플공식문서 - CAShapeLayer](https://developer.apple.com/documentation/quartzcore/cashapelayer)

[제드님 블로그 - CAShapeLayer](https://zeddios.tistory.com/824)
