---
layout: post
title: "iOS) Scenes 알아보기"
tags: [iOS, Swift]
comments: true
---

> Multiple Scenes  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

오늘은 자판기 프로젝트를 하면서 multiple scene을 지원하는 기능을 구현해야했었는데 그 당시 쏟아지는 다른 요구사항들에 밀려(?) 정확히 이해를 하지 못하고 넘어갔던 Scene 개념에 대해 정리해보려한다.

## Definition

Manage multiple instances of your app’s UI simultaneously, and direct resources to the appropriate instance of your UI.

이 문장을 읽어보면 한 앱에도 여러 인스턴스가 있을 수 있으며 이 여러 인스턴스들이 동시에 보여질 수 있다는 걸 알 수 있다. 아이폰에서는 잘 활용되지 않지만 아이패드에서 흔히 사용되는 multiple scene을 이야기하는 듯 한데 사실 최근 업데이트 이전까지는 동일한 앱을 여러 Scene으로 띄우는 것이 불가능했었다. 그런데 이제는 하나의 앱에 대해 multiple instances를 지원하면서 multiple scenes 기능이 가능해졌다. 아래사진은 메모앱을 두 Scene으로 분할하여 사용하고있는 모습이다.

![1](https://user-images.githubusercontent.com/35067611/104596850-e7a0a180-56b7-11eb-9778-683b4ac4efdc.png)

공식문서 Overview에 따르면 앱의 각 인스턴스는 UIWindowScene 객체로 관리된다고한다. 또한 하나의 scene은 앱 인스턴스의 UI를 표현하기 위해 window, viewcontroller를 포함하고 있으며 각 인스턴스에 대응되는 UIWindowSceneDelegate 객체를 갖고있다. Scene들은 concurrently run 되며 **하나의 앱에서 같은 메모리와 프로세스 공간을 공유한다.** 이러한 성질 때문에 하나의 앱에서 multiple scenes, 그리고 scene delegate 객체를 동시에 가질 수 있는 것이다.

## UIWindowScene

위에서 앱의 인스턴스는 UIWindowScene 객체로 관리된다고 했다. UIWindowScene이 무엇인지 알아보자. 공식문서에 따르면 아래와 같다.

A specific type of scene that manages one or more windows for your app.

즉 앱의 인스턴스 하나를 관리하는 타입으로 해당 인스턴스의 UI를 표현하기 위해 window를 갖고있다. scene의 상태가 변하면 scene 객체는 UIWindowSceneDelegate 프로토콜을 adopt하는 delegate 객체를 notify한다.

multiple windows 항목을 enable 해주어 간단하게 앱을 만들어서 실행해보았다. 아래와 같이 하나의 동일한 앱이지만 각자의 인스턴스에서 독립적으로 labelNumber 변수의 값이 업데이트된다.

![2](https://user-images.githubusercontent.com/35067611/104596869-ecfdec00-56b7-11eb-9bc6-30ac60ccdf41.gif)

흠.. 그런데 공식문서에서는 분명 메모리와 프로세스 공간을 공유한다고 했는데 왜 독립적으로 움직이지?  UIButton, UILable을 dump 해보면 scene이 다르면 다른 메모리공간에 참조되어있음을 알 수 있다. 그렇다면 메모리 공유는 뭘 의미하는걸까!!

![3](https://user-images.githubusercontent.com/35067611/104596870-ed968280-56b7-11eb-9847-59e70b08ed92.png)

WWDC 2019 영상에서는 AppDelegate, SceneDelegate 구조로 나뉘게 되는 iOS 13 업데이트에 대해 설명한다. 이 영상에서 말하길 scene이 사라진다면 SceneDelegate 객체가 메모리에서 해제된다고 말한다. 물론 영구적으로 삭제하는 것은 아니다. 사용자가 해당 scene으로 다시 돌아올 수도 있기 때문이다. 여기서 "scene이 사라진다"와 앱을 스와이프해서 완전히(explicitly) destroy 하는 것은 다르다. 전자의 경우 SceneDelegate의 `sceneDidDisconnect` 함수가 호출되지만 후자의 경우에는 AppDelegate의 `didDiscardSceneSessions` 함수가 호출된다.

이런 설명에서 유추해보자면 메모리공간을 "공유"한다는 말은, 두 scene에 각각 숫자를 표시하는 UILabel이 있을 때 두 변수가 동일한 메모리 공간을 차지하는것을 의미하는게 아니라 SceneDelegate 객체들이 App 단위의 하나의 메모리에서 나타나고 사라지는 scene lifecycle에 따라 관리된다는 것을 뜻하는 것 같다!

## 세 줄 요약

- Scene은 앱의 여러 인스턴스를 동시에 관리하기 위한 단위이다.
- Multiple scenes은 한 앱의 메모리와 프로세스 공간을 공유한다.
- Scene은 App 단위의 lifecycle과 별개로 scene lifecycle에 따라 관리된다.

## References

[Apple Developer Documentation - Scenes](https://developer.apple.com/documentation/uikit/app_and_environment/scenes)

[Apple Developer Documentation - UIWindowScene](https://developer.apple.com/documentation/uikit/uiwindowscene)

[Architecting Your App for Multiple Windows - WWDC 2019 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2019/258/)
