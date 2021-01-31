---
layout: post
title: "iOS) iOS 앱 번들 구조와 샌드박스 방식"
tags: [iOS]
comments: true
---

> App Bundle & SandBox 구조  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

## iOS 앱 번들 구조

### 번들이란 무엇인가?

애플 공식문서에 따르면 번들은 실행가능한 코드와 관련 리소스(이미지, 사운드 등)을 한 공간에 묶는 파일시스템에 있는 `디렉토리`이다. iOS나 OS X 환경에서는 애플리케이션, 프레임워크, 플러그인, 그리고 다른 타입의 소프트웨어들이 번들이라고 할 수 있다. 즉, 번들이란 실행가능한 코드와 코드에 의해 사용되는 리소스를 갖는 standardized, hierarchical 디렉토리 구조이다.

### Application Bundles

개발자들에 의해 생성되는 가장 흔한 번들은 Application Bundles이다. 애플리케이션 번들은 앱을 성공적으로 실행시키기 위한 모든 것을 저장한다. 물론 더 상세한 세팅이나 구조는 플랫폼(iOS, macOS 등)에 따라 달라지지만 번들을 사용하는 방법은 동일하다.

먼저 애플리케이션 번들에는 어떤 파일들이 들어있는지 보자.

- **Info.plist** 파일: 앱을 위한 컨피규레이션 정보를 담은 런타임 컨피규레이션 파일.
- **Executable**: 모든 앱은 반드시 실행가능한 파일이 필요하다. 앱의 main entry point와 앱 타겟과 정적으로 link된 모든 코드를 포함한다.
- **Resource files**: Executable file 밖에 있는 데이터들을 의미한다. 보통 이미지, 아이콘, 사운드, nib 파일, 문자열 파일, 컨피규레이션 파일, 데이터 파일 등을 포함한다. 모든 리소스 파일들은 localized 혹은 nonlocalized 될 수 있으며 localized 된 파일이라면 Resources 서브디렉토리 내에 대응하는 언어나 locale정보에 `lproj` 확장자를 가진 파일로 저장된다.
- **Other support files**: 맥의 앱에는 추가적인 high-level 리소스(예를 들어 private framework, plug-in, 다큐먼트 템플릿, 커스텀 데이터 리소스 등)을 추가할 수 있다. iOS에서는 custom data resource를 포함하는 것이 가능하지만 custom framework, plug-ins는 불가능하다.

애플리케이션 번들 중 리소스는 거의 대부분 옵셔널하지만 항상 그런 것은 아니다. 예를 들어 iOS 앱은 보통 추가적인 아이콘 이미지를 요구하기도한다.

보통 iOS 앱의 애플리케이션 번들은 실행파일(executable)과 앱에 의해 사용되는 리소스(아이콘, 이미지, localized 콘텐트 등)을 최상위 번들 디렉토리에 포함한다. 아래는 MyApp이라는 앱의 구조이다. 서브 디렉토리로 존재하는 파일들은 localized를 위한 파일들이다. 하지만 얼마든지 서브 디렉토리는 개발자가 직접 생성하고 관련된 파일끼리 리소스를 관리하는 것도 가능하다.

```swift
MyApp.app
   MyApp
   MyAppIcon.png
   MySearchIcon.png
   Info.plist
   Default.png
   MainWindow.nib
   Settings.bundle
   MySettingsIcon.png
   iTunesArtwork
   en.lproj
      MyImage.png
   fr.lproj
      MyImage.png
```

위 번들 구조에 포함된 콘텐트를 하나씩 뜯어보자.

- MyApp: 실제 앱 이름에서 `.app` 확장자를 뗀 것이 곧 실행파일(executable)이다. 번들 구조에 반드시 있어야 한다.
- Application icons: MyAppIcon.png, MySearchIcon.png, MySettingsIcon.png이 여기에 속한다. 아이콘은 알려져있듯 앱을 나타내는데 사용된다. 홈 화면에서, 혹은 앱을 검색했을 때에 아이콘이 가장 먼저 보여지는 정보이므로 꼭 포함해야한다.
- Info.plist: 번들 ID, 버전 넘버, 디스플레이 네임 등과 같은 앱의 컨피규레이션 정보를 담고 있는 파일로 꼭 포함되어야 한다.
- Launch Images: 위 구조에서는 Default.png이며 앱의 initial interface를 보여주는 이미지이다. 시스템은 제공된 런치 이미지 중 하나를 앱이 윈도우와 유저 인터페이스를 로드할 동안 임시 배경으로 사용한다. 만약 런치 이미지가 없다면 검은 화면이 보여질 것이다.
- MainWindow.nib: 앱의 main nib file은 런치타임에 앱을 로드하기 위한 기본 인터페이스 오브젝트를 포함한다. 보통 앱의 메인 윈도우 객체와 앱 델리게이트 객체를 갖고 있다.
- Settings.bundle: 앱의 specific preferences를 포함하는 특별한 타입의 플러그인이다. 이 번들은 preferences를 configure하고 디스플레이하기 위한 property list와 다른 리소스 파일들을 포함한다.
- Custom resource files: non-localized 리소스들은 탑 레벨 디렉토리에, localized 리소스는 language-specific 서브 디렉토리에 위치한다.

잠시 정리를 해보자. 번들은 결국 하나의 앱을 구성하는 여러가지 요소(실행파일, 리소스, 컨피규레이션 파일 등)를 한 곳에 묶어서 관리하는 하나의 디렉토리, 모음이다. 그리고 이 모음에는 필수적으로 포함되어야 하는 것과 필수는 아니지만 권장되는 여러 요소들이 있고 각각의 요소들은 저마다의 역할이 있다.

## 애플의 샌드박스 방식

### SandBox

이번엔 애플의 샌드박스 방식에 대해 알아보자. 샌드박스(Sandbox)란 직역하면 모래통인데, 어린아이를 보호하기 위해 이 샌드박스 내에서만 놀도록 하는 의미에서 유래된 보안 모델이다. iOS는 기본적으로 앱마다 별도의 파일을 생성해서 공유되지 않도록 하고 외부로부터 들어온 접근에 대해서 보호되는 영역으로 시스템이 부정적으로 조작되는 것을 막는 보안 형태를 갖고있다. 샌드박스가 있고 없고의 차이점은 아래 사진에서 쉽게 이해할 수 있다.

![1](https://user-images.githubusercontent.com/35067611/104592553-b7ee9b00-56b1-11eb-80f8-692e309941b9.png)

왼쪽 그림처럼 앱이 샌드박스화 되지 않는다면 앱은 실행하는 사용자의 모든 권한을 갖게되고 사용자가 접근할 수 있는 모든 리소스에 동일하게 접근이 가능하다. 이럴 경우, 앱이나 연결된 프레임워크에 보안적 결함이 있을 때 공격자는 해당 앱을 제어하기 위해 취약점을 악용할 수 있으며 사용자가 수행할 수 있는 모든 작업을 수행할 수 있게 된다. 이러한 문제를 해결하기 위해 애플은 앱을 샌드박스화 하는 것이다.

한마디로 하나의 앱은 하나의 샌드박스 내에서만 놀 수 있는 것이다. 이런 보안상의 이유로 iOS는 앱이 모두 SandBox화 되어있다. 즉, **`샌드박스`는 파일, 환경설정, 네트워크 리소스, 하드웨어 등에 대한 앱의 접근을 제한하는 세분화된 제어 집합, 커널 수준에서 시행되는 접근 제어 기술**이라고 할 수 있다. 이렇게 관리함으로써 데이터를 분리하여 고립시키고, 보안 침해의 가능성을 낮출 수 있다.

애플의 샌드박스 전략은 크게 두 가지이다.

1. 앱 샌드박스를 사용하면 앱이 시스템과 상호작용하는 방식을 설명할 수 있다. 시스템에서 작업을 완료하는 데 필요한 접근권한을 앱에 `부여`한다.
2. 앱 샌드박스를 사용하면 열기 및 저장, 드래그 앤 드롭 등 친숙한 사용자 상호작용을 통해 앱에 투명하게 추가 접근 권한을 `부여`할 수 있다.

![2](https://user-images.githubusercontent.com/35067611/104592564-bb822200-56b1-11eb-877c-174f664fd1f8.png)

이렇게 샌드박스화 시킴으로써 하나의 앱은 그 앱 안에 있는 데이터 이외에는 접근할 수 없게된다. 하지만 사용자 데이터는 앱 외부에 있다. 예를 들어 사진, 연락처 등 앱 외부에 있는 데이터는 앱이 접근할 수 없게 되는 것이다. 하지만 우리는 지금까지 이런 앱 외부 데이터를 어렵지 않게 앱에서 끌어다 썼다. 이것을 가능케 하는 것이 "접근 권한"을 사용자가 명시적으로 허용했을 때에 최소한으로 접근할 수 있도록 하는 구조이다.

## 세 줄 요약

- 번들은 하나의 앱을 구성하는 여러 요소를 묶음으로 관리하는 디렉토리이다.
- 샌드박스는 앱의 외부 데이터에 대한 접근을 제한하는 보안 모델이다.
- 지금까지 접근권한허용 메세지들이 귀찮고 왜 굳이 띄우나? 라는 생각을 했지만 반성해야겠다..

## References

[Bundle Structures](https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFBundles/BundleTypes/BundleTypes.html#//apple_ref/doc/uid/10000123i-CH101-SW1)

[[iOS] 앱 샌드박스(App Sandbox)와 Container Directory](https://jinnify.tistory.com/26)

[iOS ) Apple의 Sandbox정책과 Files앱](https://zeddios.tistory.com/432)
