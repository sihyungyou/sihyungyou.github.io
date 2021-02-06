---
layout: post
title: "iOS) [번역] Using Delegates to Customize Object Behavior"
tags: [iOS, Swift]
comments: true
---

> Cocoa Design Pattern - Delegates  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

# **Overview**

앱 내에서 이벤트를 알려주는(inform) Coca objects와 상호작용(interact) 하기 위해 delegate을 사용한다.

# **Adopt a Delegate Protocol**

Cocoa APIs는 delegate 메소드를 포함하는 프로토콜을 제공한다. 이벤트가 발생하면 (예를 들어 사용자가 window를 resizing 하는 등의) delegator 클래스는 해당 이벤트를 감지(detect) 하고 delegate로 지정(specify)해놓은 클래스의 delegate 메소드를 호출한다. delegate 메소드는 앱이 해당 이벤트에 대해 어떻게 반응(respond)할 지 정의할 수 있다.

아래의 예제는 `NSWindowDelegate` 프로토콜을 채택하고 해당 프로토콜의 `window(_:willUseFullScreenContentSize:)` 메소드를 구현하고 있다.

```swift
class MyDelegate: NSObject, NSWindowDelegate {
    func window(_ window: NSWindow, willUseFullScreenContentSize proposedSize: NSSize) -> NSSize {
        return proposedSize
    }
}
```

# **Check That Delegates Exist**

Cocoa delegation pattern은 delegates가 instantiated 되도록 요구하지 않는다. 만약 이벤트에 대해 반응할 필요가 없다면 delegate을 생성하지 않아도 된다. 객체의 delegate에서 메소드를 호출하기 전에 delegate이 `nil`이 아닌 것을 확인하라.

아래 예제는 `NSWindow`를 생성하고 delegate에게 메세지를 보내기 전에 옵셔널 체이닝을 통해 window의 delegate이 존재하는지 확인한다.

```swift
let myWindow = NSWindow(
    contentRect: NSRect(x: 0, y: 0, width: 5120, height: 2880),
    styleMask: .fullScreen,
    backing: .buffered,
    defer: false
)

myWindow.delegate = MyDelegate()
if let fullScreenSize = myWindow.delegate?.window(myWindow, willUseFullScreenContentSize: mySize) {
    print(NSStringFromSize(fullScreenSize))
}
```

# References  
[Apple Developer Documentation - Using Delegates to Customize Object Behavior](https://developer.apple.com/documentation/swift/cocoa_design_patterns/using_delegates_to_customize_object_behavior)
