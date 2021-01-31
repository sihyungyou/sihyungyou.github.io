---
layout: post
title: "iOS) View & Window 알아보기"
tags: [iOS, Swift]
comments: true
---

> 뷰와 윈도우는 어떻게 다를까  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

Frame, bounds가 View의 위치와 크기에 대해 다루는 property였다면 이번엔 View 자체에 대해 알아보자. 더불어 View 계층 구조의 루트라고 할 수 있는 Window에 대해서도 함께 공부해보즈아! (사실 이 둘은 SceneDelegate, AppDelegate를 공부하기 위한 초석느낌이다)

## Definition

### View

역시 애플문서로 시작해보면 `View`는 아래와 같이 정의된다.

The view that the controller manages.

음.. 너무 불친절한데..? 아무튼 view는 UIView의 instance property다. 그래서 UIView의 정의도 찾아봤다.

An object that manages the content for a rectangular area on the screen.

Overview에서 속시원히 말해주는데 "Views are the **fundamental building blocks** of your app's user interface", "and the UIView class defines the behaviors that are common to all views" 라고 한다. 결국 "우리가 보는 휴대폰의 사각형 화면 속 content를 관리하는 단위의 객체" 정도로 생각하면 될 듯 하다.

공식문서 Discussion에 따르면 ViewController의 View에 접근했는데 nil이라면 (default가 nil임) ViewController가 자동으로 `loadView()` 메소드를 호출하여 View를 반환해준다고 한다.  한가지 더 기억해야할 것은 하나의 view object가 여러 view controller object와 연결되면 안된다는 것이다. 이 규칙이 허용되는 예외는 container view controller를 사용할 때 라고한다. (아직 안 와닿네..)

### Window

`window`도 역시나 쏘 불친절..

The receiver’s window object, or nil if it has none.

참고로 window는 UIView가 아니라 UIWindow의 instance property라서 UIWindow는 뭐하는 친구인지 찾아봤다.

The backdrop for your app’s user interface and the object that dispatches events to your views.

한결같이 한 줄을 넘기지 않네..? Anyway! Overview 파트로 내려가면 좀 자세히 알려준다. 주의해서 볼 점이 몇 가지 있었다.

먼저 window는 ViewController와 함께 event handling, performing tasks 등 기본적인 애플리케이션의 operation을 수행한다. 이 window는 앱의 컨텐츠를 메인 스크린에 보여주며 하나의 앱에서는 하나의 window만 있으면 된다. 물론 main screen을 여러개 띄우고 싶다면 window를 더 생성할 수도 있다. (이건 SceneDelegate을 다룰 때 다시 보게된다)

두번째로 "Windows **do not have any visual appearance** of their own. Instead, a window hosts one or more views, which are managed by the window's root view controller." window는 실제로 눈에 보이는 실체를 갖고 있는게 아니라 root view controller를 관리할 뿐이라는 것! 이렇기에 UIWindow를 subclassing 해야 할 경우는 거의 없어야 한다. 왜 와이? view controller에서 웬만한 구현은 더 쉽게 가능하므로! 단 subclassing이 필요한 경우가 있는데 `becomeKey()`, `resignKey()` 메소드를 호출해야할 때다. (이건 나중에 UIScreen 부분에서 따로 정리하는걸로..)

아무튼 다시 정리를 해보자면 view는 "우리가 보는 휴대폰의 `사각형 화면 속 content를 관리`하는 단위의 객체", window는 "그런 `view들을 보여주는 컨테이너` 역할을하는 객체" 정도 되겠다.

+ 모든 View는 Window로 묶여있다는 것!

+ UIWindow가 UIView의 하위클래스이므로 Window는 그 자체가 View라고 할 수 있음!

## 좀 더 파보즈아

### 사진으로 이해해보기

![1](https://user-images.githubusercontent.com/35067611/104596580-82e54700-56b7-11eb-8012-cc4915a7edbb.jpeg)

위 사진에서 hierarchy를 보면 View가 UIWindow 하위에 들어가있는 것을 볼 수 있다. 그리고 그 View 하위에도 여러 view들(ImageView, Button, ToolBar 등)이 subview로 계층화되어있다.

![2](https://user-images.githubusercontent.com/35067611/104596595-8547a100-56b7-11eb-81db-4f86dbd5003f.png)

![3](https://user-images.githubusercontent.com/35067611/104596599-8547a100-56b7-11eb-8125-cd81d3f7c7a0.png)

위 사진들은 frame, bounds 공부할 때 만든 프로젝트의 View Hierarchy를 debug 찍어본 것이다. 자세히보면 UIWindow 위에 또 WindowScene이라고 있는데.. 이것도 다음에 따로 공부해봐야겠다. 파도파도 계속 나오는 즐거운 iOS!

### Most Apps need only one window...

앱은 반드시 하나의 window가 필요하다고 했다. 지금까지 나는 window를 만든 적이 없는데..? (그래도 잘 돌아갔음) 왜 와이!? SceneDelegate 에서 자동으로 하나 만들어줬기 때문! 두둥-탁!

![4](https://user-images.githubusercontent.com/35067611/104596600-85e03780-56b7-11eb-8f6f-7c38c9e672a4.png)

여러 window가 필요한 경우도 있을텐데 이는 SceneDelegate, AppDelegate 공부할 때 자세히 알아보도록 하자!

### 어디에 쓰이는가?

UIWindow가 하는 일:

- Setting the z-axis level of your window, which affects the visibility of the window relative to other windows.
- Showing windows and making them the target of keyboard events.
- Converting coordinate values to and from the window’s coordinate system.
- **Changing the root view controller of a window.**
- Changing the screen on which the window is displayed.

지금까지의 얕은 개발경험을 토대로 가장 중요하게 작용했던 UIWindow의 역할은 4번이었다. 음식 주문앱 프로젝트를 만들 때 회원가입 여부에 따라 상품목록화면/회원가입화면을 root view controller로 정해줘야했기 때문!

UIView가 하는 일:

- Drawing and animation
    - Views draw content in their rectangular area using UIKit or Core Graphics.
    - Some view properties can be animated to new values.
- Layout and subview management
    - Views may contain zero or more subviews.
    - Views can adjust the size and position of their subviews.
    - Use Auto Layout to define the rules for resizing and repositioning your views in response to changes in the view hierarchy.
- Event handling
    - A view is a subclass of `UIResponder` and can respond to touches and other types of events.
    - Views can install gesture recognizers to handle common gestures.

UIView의 역할은 워낙 눈에 보이는 모든 behavior, content와 직접적이라서 거의다 사용해본 것 같다.

## 세 줄 요약

- View는 화면에 보이는 UI요소를 관리하는 가장 기본적인 단위이며 window는 그런 view들을 묶어놓는 상위 view다.
- Window는 실제로 화면에 보이지 않으며 웬만한 앱은 하나의 window만 필요로 한다.
- Window는 SceneDelegate에서 자동으로 생성해주므로 직접 선언해서 사용할 필요가 없다.

## References

[Apple Developer Documentation - view](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621460-view)

[Apple Developer Documentation - UIView](https://developer.apple.com/documentation/uikit/uiview)

[Apple Developer Documentation - window](https://developer.apple.com/documentation/uikit/uiview/1622456-window)

[Apple Developer Documentation - UIWindow](https://developer.apple.com/documentation/uikit/uiwindow)
