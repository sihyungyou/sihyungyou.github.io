---
layout: post
title: "iOS) Delegate 패턴"
tags: [iOS]
comments: true
---

> Design Pattern  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

Delegate 패턴은 애플이 제공하는 많은 API(테이블뷰, 컬렉션뷰 등)에서도 사용되는 디자인 패턴이다. "Delegate"라는 단어 자체는 위임자, 혹은 대리자라는 뜻인데 옵저버 패턴과 비교하면 이름이 확 와닿지는 않는다. 오늘은 Delegate 패턴은 무엇이며 어떻게 설계되고 왜 사용되는지에 대해 알아보자

## Delegate의 의미

Delegate패턴에서의 Delegate는 "어떤 객체가 해야 하는 일을 부분적으로 `확장`해서 `대신 처리`를 한다"는 의미로 받아들이면 좋을 것 같다.

## 실제 구현되는 방식

옵저버 패턴에서는 관찰대상과 관찰자로 구분하여 생각했다면 Delegate 패턴에서는 `대신 처리해줄 객체`와 `처리하라고 시키는 객체`로 나누어 생각해보자. 테이블뷰를 예로 들어보면, 뷰컨트롤러가 대신 처리해줄 객체이고 테이블뷰가 처리하라고 시키는 객체이다.

아래는 흔히 사용되는 뷰 컨트롤러에서 extension을 활용해 UITableViewDataSource 프로토콜에 선언된 함수를 정의하는 코드이다.

```swift
self.myTableView.dataSource = self // 여기서 dataSource가 delegate, 즉 "대리자"다.
```

```swift
extension TableViewController: UITableViewDataSource {
  func tableView(_ tableView: UITableView,
                  numberOfRowsInSection section: Int) -> Int {
    return self.items.count
  }

  func tableView(_ tableView: UITableView,
                  cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell: UITableViewCell = tableView.dequeueReusableCell
                    (withIdentifier: "TableViewCell")! as UITableViewCell

    cell.textLabel?.text = items[indexPath.row]

    return cell
  }
}
```

이렇게 코드를 짜고 나면

- 테이블뷰라는 객체에 속한 dataSource라는 변수(UITableViewDataSource 프로토콜 타입)에 뷰컨트롤러가 대입된 형태이고(이게 가능한 이유는 뷰컨트롤러가 해당 프로토콜을 상속받아 형변환이 되기 때문), 테이블뷰에서는 특정 behavior에 대해서 dataSource 변수를 통해 적절한 함수를 호출할 것이다.
- 이 때 호출되는 함수는 UITableViewDataSource 프로토콜에 선언된 함수일텐데, 이 함수를 현재 뷰컨트롤러가 프로토콜을 채택하여 자기만의 behavior로 재정의하고 있는 것과 같다.
- 그리고 UITableViewDataSource 프로토콜에 선언된 함수를 호출하기 위해 테이블뷰 내부에서는 dataSoruce.함수() 와 같이 delegate 변수를 사용하게된다.
- 그런데 dataSource라는 변수에는 위에서도 말했듯 뷰컨트롤러가 대입되어있으므로 뷰컨트롤러.함수() 와 같이 작동하게 되는 것이다.
- 이 과정을 거치면 뷰컨트롤러는 자신을 위임자(delegate 변수)에게 넘겨주고 위임자는 뷰컨트롤러(수신자)의 메소드를 `대신` 실행해주는 형태가 된다.

## 왜 프로토콜이어야 하는가?

그런데 여기서 위임자 변수가 프로토콜 타입이라고 했다. 클래스면 안될까? 물론 된다! 하지만 확장성에서 매우 불리한데 상속을 받아야하기 때문에 그렇다.

예를 들어 뷰컨트롤러를 대입할 때 (tableView.delegate = self 와 같이) delegate 변수가 프로토콜이 아닌 클래스라면 반드시 뷰컨트롤러가 해당 클래스와 동일하거나 상속을 받아야한다는 뜻이다. 하지만 다중 상속도 불가능하고 프로토콜 채택이 훨씬 가볍기 때문에 확장성에서 유리하다. 프로토콜을 채택하면 채택한 프로토콜로 형변환이 가능해지기 때문에 Swift에서는 Delegate 패턴을 프로토콜을 사용해서 구현한다.

또한 클래스 객체를 통째로 넘겨버리면 굳이 처리해야할 부분 이외의 부분까지 한번에 넘어가게 된다. 물론 나머지 부분은 건들지 않을 수는 있지만 이는 여전히 버그를 발생시킬 여지가 남아있기 때문에 지양하는게 좋다.

## Why Delegate Pattern?

그렇다면 왜 Delegate 패턴을 사용하는가? Notification Center, 옵저버 패턴 등을 통해서도 클래스 간의 상호작용을 구현할 수 있지 않을까?

- Delegate 패턴은 클래스 간 요구사항을 전달해주는 프로토콜만 있다면 연결이 수월해진다.
- 클래스나 구조체를 상속할 필요가 없으므로 더욱 가볍게 사용할 수 있다.
- Delegate 패턴은 1:1 관계에서, 옵저버 패턴은 1:N 또는 N:N 관계에서 더욱 적절하다.
- 프로토콜을 준수하는 것만으로도 구현이 가능하므로 매우 유연하다.

## 단점은 없을까

프로토콜을 사용한다는 점에서 명확한 장점을 갖지만 동일한 이유로 단점도 생긴다. 프로토콜은 결국 하나의 약속이고 "정해진 메소드만" 구현하도록 권장된다. 그러므로 상속한 프로토콜을 프로퍼티로 갖고 있게 되면 해당 프로퍼티에서는 프로토콜에서 `약속된 메소드만 실행가능`하다는 단점이 있다. 만약 delegate 패턴으로만 설계를 하려고 한다면 위임자 변수를 점점 늘리고 프로토콜을 여러개 만들어야할 것이다.

## References

[iOS Delegate 패턴에 대해서 알아보기](https://magi82.github.io/ios-delegate/)

[[Swift] - Delegate Pattern에 대해서](https://velog.io/@iwwuf7/Swift-Delegate-Pattern에-대해서)
