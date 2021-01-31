---
layout: post
title: "iOS) 앱/Scene 생명주기"
tags: [iOS]
comments: true
---

> App/Scene Life Cycle  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

## 앱 생명주기란 무엇인가?

앱 생명주기란 앱의 실행부터 종료까지의 주기를 말하며, 시스템 알림에 응답하고 여러 시스템 관련 이벤트를 처리하는 단계들을 말한다. 이와 비슷하게 뷰 컨트롤러 생명주기(viewDidLoad, viewWillAppear)도 있다.

## 왜 중요한가?

CPU는 한정된 리소스로 사용자에게 여러가지 기능을 제공한다. iOS 모바일 환경 또한 하나의 CPU가 제한된 메모리와 시간을 여러 앱에 할당하게된다. 이 때 가장 높은 우선순위를 갖는 앱은 한 시점에 사용자에게 보여지는 앱일 것이다. 그런데 여러 앱을 번갈아가며 사용할 때 잠시 다른 앱을 사용한다고해서 이전의 모든 데이터가 초기화되거나 다시 로그인을 해야하는 등의 흐름은 사용성을 매우 해칠 것이다. iOS 개발자로서 앱 생명주기에 대해 이해해야 앱이 백그라운드로 갔을 때 데이터를 저장해서 사용성을 높인다든지, 은행 앱 같은 보안이 중요한 앱의 경우 로그인을 일부러 다시 하도록 하는 등의 처리를 올바르게 할 수 있을 것이다.

## Main Run Loop

먼저 사용자의 터치 이벤트를 받는 시점부터 코드로 구현된 여러 로직들이 돌아가는 시점까지의 흐름을 보자.

![1](https://user-images.githubusercontent.com/35067611/104591249-bfad4000-56af-11eb-8d43-b8f47bcc9098.png)

위 사진에서 볼 수 있듯, 앱의 Main Run Loop는 모든 사용자가 발생시키는 이벤트에 따라 처리된다. 이벤트가 발생했을때, UIKit에 의해 설정된 Port를 통해 내부의 Event queue에 이벤트를 담아놓고, 담겨있는 이벤트를 Main Run Loop에서 하나하나씩 실행한다.

그런데 이렇게 main loop를 실행하기 전, 즉 사용자가 앱에 어떠한 이벤트를 주지는 않았지만 앱 아이콘을 눌러서 앱을 실행시킨 시점에 추가적으로 하는 일이 있다. 바로 UIApplication 생성과 @UIApplicationMain 어노테이션이 붙은 클래스를 찾아 AppDelegate를 생성하는 일이다.

## UIApplication, AppDelegate

UIApplication 객체는 싱글톤으로 Event Loop에서 발생하는 이벤트들을 감지하고 Delegate에 전달하는 역할을 한다. 이렇게 UIApplication 객체로부터 전달되는 메세지를 AppDelegate 객체가 받고 각각의 상황(예를 들어 백그라운드로 간다든지, 메모리가 무족하여 경고 메세지가 왔다든지 등)에서 실행할 함수들을 정의한다. 여기서 말하는 "상황"이 오늘 메인 주제인 앱 생명주기와 관련이 있으며 이 `상황(혹은 상태)`은 아래와 같이 다섯가지로 구분될 수 있다.

```
Not Running: 앱이 실행되지 않은 상태
Inactive: 앱이 실행중인 상태 그러나 아무런 이벤트를 받지 않는 상태
Active: 앱이 실행중이며 이벤트가 발생한 상태
Background: 앱이 백그라운드에 있는 상태 그러나 실행되는 코드가 있는 상태
Suspened: 앱이 백그라운드에 있고 실행되는 코드가 없는 상태
```

이렇게 다섯가지 상태가 있고 각각의 상태에서 실행할 `함수`들을 정의할 수 있다고 했다. 그것이 바로 AppDelegate.swift 파일에서 우리가 볼 수 있었던 delegate 함수들이다.

```
application(_:didFinishLaunching:) - 앱이 처음 시작될 때 실행
applicationWillResignActive: - 앱이 active 에서 inactive로 이동될 때 실행
applicationDidEnterBackground: - 앱이 background 상태일 때 실행
applicationWillEnterForeground: - 앱이 background에서 foreground로 이동 될때 실행 (아직 foreground에서 실행중이진 않음)
applicationDidBecomeActive: - 앱이 active상태가 되어 실행 중일 때
applicationWillTerminate: - 앱이 종료될 때 실행
```

이 모든 함수를 다 정의할 필요는 없고 UIApplicationDelegate 객체 내부를 보면서 정의할 수 있는 함수 중에 필요한 것만 쓰면 된다. 실제로 AppDelegate에서 func application만 쳐도 아래에 정의할 수 있는 delegate 함수들의 목록을 자동완성기능으로 확인할 수 있다.

![2](https://user-images.githubusercontent.com/35067611/104591258-c3d95d80-56af-11eb-96e5-497be31a5277.png)

<img width="993" alt="3" src="https://user-images.githubusercontent.com/35067611/104591261-c50a8a80-56af-11eb-95c3-030950281a5e.png">

## 다섯가지 상태에 대해서

방금 위에서 언급한 다섯가지 상태에 대해 조금 더 실제로 언제 어떤 상태에 해당하는지 살펴보자면 아래와 같다.

`Not Running` : 아무것도 실행하지 않은 상태. (또는 실행중이긴 하지만 시스템에 의해서 종료된 상태)

`Inactive` : 앱이 foreground 상태로 돌아가지만 이벤트는 받지 않는 상태. 앱의 상태 전환 과정에서 잠깐 머물거나 앱을 사용하다가 알람과 같은 인터럽트가 발생해서 잠시 유저이벤트를 받을 수 없는 상태.

`Active` : 일반적으로 앱이 돌아가는 상태 (이벤트를 받는 단계)

`Background` : 앱이 suspended 상태로 진입하기 전 거치는 상태

`Suspended` : 앱이 background에 있지만 아무 코드도 실행하지 않는 상태.

여기서 조금 헷갈리는 것이 background와 suspended의 차이였다. Background 상태에서도 앱이 실행될 수 있다는 것을 이해하면 도움이 되는데 예를 들어 음악 재생, GPS, 녹음 등의 기능은 앱을 끄거나 다른 앱으로 넘어가도 계속 실행된다.

하지만 suspended 상태는 그렇지 않다. 더해서, suspended 상태에서는 앱이 memory에는 유지되지만 memory 부족 상태가 나타나면 system이 memory에서 제거한다는 차이점이 있다. 만약 background에서 특별히 수행해야 하는 기능이 없다면 background 상태를 거쳐서 suspended 상태로 넘어간다.

## SceneDelegate

잠깐 정리를 해보자면 앱 생명주기에는 다섯가지 상태 (not running, inactive, active, background, suspend)가 있고 각각의 상태에 실행되는 함수를 AppDelegate에서 UIApplicationDelegate 프로토콜을 채택하여 정의할 수 있다 (applicationDidFinishLaunching, applicationDidBecomeActive 등). 즉, AppDelegate는 앱의 생명주기 중 여러 단계에서 해야할 일들을 개발자가 코드로 정의해줄 수 있도록 도와주는 객체이다.

그렇다면 SceneDelegate은 왜 필요한 것일까? 앱 생명주기는 AppDelegate에서 컨트롤해주고 뷰컨트롤러 생명주기는 ViewController에서 적절히 처리할 수 있으면 된 것 아닐까? SceneDelegate는 iOS 13 이후부터 지원되는 multiple scene 기능에 잘 대응하기 위해 존재한다.

## Scene, Multiple Scenes

Multiple Scenes 기능과 Scene 생명주기를 다루기에 앞서 Scene이라는 단어를 사용할 때 짚고 넘어가야하는 것이 있다. 바로 스토리보드를 사용할 때 언급되는 Scene과는 의미가 다르다는 점을 이해해야하는데, 스토리보드에서는 각 화면을 하나의 Scene이라고 하지만 Multiple Scenes 기능에서 말하는 Scene은 하나의 앱이 여러 분신처럼 동시에 사용되는 경우 각각의 분신을 의미한다. 즉, Scene은 앱 UI의 하나의 복사본 또는 앱 UI의 하나의 인스턴스라고 생각하면 된다. 이해를 돕기 위해 아래 사진을 보자.

![4](https://user-images.githubusercontent.com/35067611/104591267-c63bb780-56af-11eb-90ad-2d772605dd3a.png)

동일한 메모 앱이 두 화면으로 나뉘어(split) 사용되고 있는데 이 때 각각을 Scene이라고 하며 이러한 기능이 iOS 13부터 지원되었다. 그리고 이 각각의 Scene은 당연히 자신만의 생명주기가 필요하므로 이를 잘 컨트롤하기 위해 AppDelegate에 더해 SceneDelegate가 추가된 것이다!

Scene-based 라이프 사이클도 앱 생명주기와 크게 다르지 않다. 여러 상태가 있고 각 상태에서 아래 사진에 명시된 delegate 함수들이 호출되며 이 함수들을 override 할 수 있다.

![5](https://user-images.githubusercontent.com/35067611/104591272-c76ce480-56af-11eb-9fbe-cc97f5bae254.png)

## App, Scene 생명주기 한눈에 보기

지금까지 알아본 생명주기 각각의 상태, delegate 함수, 흐름을 한 그림에 나타내면 아래와 같다.

![6](https://user-images.githubusercontent.com/35067611/104591274-c8057b00-56af-11eb-8cf1-7501743b96ba.png)

## References

[앱 생명주기(App Lifecycle) vs 뷰 컨트롤러 생명주기(ViewController Lifecycle) in iOS](https://medium.com/ios-development-with-swift/앱-생명주기-app-lifecycle-vs-뷰-생명주기-view-lifecycle-in-ios-336ae00d1855)

[Application Lifecycle(어플리케이션 라이프 사이클)](https://billions-and-billions.tistory.com/9)

[[App's Life Cycle] App-Based Life-Cycle과 Scene-Based Life-Cycle](https://eunjin3786.tistory.com/163)

[오늘의 Swift 상식 (앱 생명주기)](https://medium.com/@jgj455/오늘의-swift-상식-앱-생명주기-878dfe51d182)
