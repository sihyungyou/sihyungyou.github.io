---
layout: post
title: "iOS) Auto Layout 설정하기 (programmatically)"
tags: [iOS, Swift]
comments: true
---

> 코드로 레이아웃을 잡아보자  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

오토 레이아웃이란 "기기의 화면 크기가 변해도 사용자 입장에서 뷰의 비율이 동일하게끔 보이도록 배치하는 것"이다. 주로 storyboard를 이용해서 뷰를 배치할 때 constraint를 적용시켜서 이 레이아웃을 잡아주곤 하는데, 이 과정은 코드로도 동일하게 구현이 가능하다. 오늘은 코드로 오토 레이아웃을 잡는 방법에 대해 알아보자.

## 1. translatesAutoresizingMaskIntoConstraints 옵션 끄기

먼저 이 옵션을 꺼야 추가적인 constraint를 정의할 수 있다. 이 옵션 자체에 대해 설명하기 위해서는  autoresizing, constraint 등 부가적으로 생각할 것들이 있는데 아예 다른 포스팅으로 올리도록 하겠다. 암튼 false로 이 옵션을 꺼주자!

```swift
yellowView.translatesAutoresizingMaskIntoConstraints = false
```

## 2. addSubView

constraint를 설정해주기 전에 먼저 subview로 추가해주어야한다.

```swift
self.view.addSubview(yellowView)
```

## 3. constraints 추가

이제 본격적으로 constraints를 추가해보자. 이 때 두 가지 방법으로 추가해줄 수 있다. 먼저 한 constraint 씩 모두 `.isActive = true`를 켜주는 방법이다.

```swift
yellowView.trailingAnchor.constraint(equalTo: self.view.trailingAnchor, constant: -30).isActive = true
yellowView.centerYAnchor.constraint(equalTo: self.view.centerYAnchor).isActive = true
yellowView.widthAnchor.constraint(equalToConstant: 150).isActive = true
yellowView.heightAnchor.constraint(equalToConstant: 150).isActive = true
```

이게 좀 귀찮다 싶으면 constraints의 배열을 하나 선언해서 NSLayoutConstraint에 넘겨주는 방법도 있다.

```swift
let indigoViewConstraints = [indigoView.leadingAnchor.constraint(equalTo: yellowView.leadingAnchor),
                             indigoView.bottomAnchor.constraint(equalTo: yellowView.topAnchor, constant: -30),
                             indigoView.widthAnchor.constraint(equalToConstant: 150),
                             indigoView.heightAnchor.constraint(equalToConstant: 150)]

NSLayoutConstraint.activate(indigoViewConstraints)
```

## 결과

위의 세 단계를 거치면 아래 사진과 같이 예쁘게 레이아웃을 잡을 수 있다. 참고로 왼쪽의 두 뷰는 interface builder로, 오른쪽의 뷰들은 코드로 잡아주었다.
![1](https://user-images.githubusercontent.com/35067611/105939693-10a03980-609d-11eb-8db6-3b1f1df74192.png)

## 값 지정 시 유의사항

그런데 위 코드를 자세히 보면 constraint를 잡아줄 때 constant에 음수 값이 들어가는 것을 볼 수 있다. 이는 단순히 "간격"을 지정해주는 것이 아니라 좌표계 기준으로 방향까지도 정해줘야하기 때문이다. 방향에 따라 양수, 음수 여부는 아래 그림과 같이 정해진다.

![3](https://user-images.githubusercontent.com/35067611/105939713-1564ed80-609d-11eb-8f41-61dcdbc6028c.png)

그림의 파란 사각형을 `blueView`라고 하자. blueView 왼쪽에 `yellowView`를 추가하고 싶다. 이 때 yellowView의 trailing을 blueView의 leading으로부터 30만큼 떨어뜨리고 싶다면 방향을 고려해 -30 값을 줘야한다는 뜻이다. 혹은 yellowView를 blueView 위에 30만큼 떨어뜨려서 위치시키고 싶다면 yellowView의 bottom을 blueView의 top으로부터 -30만큼 떨어뜨려야한다.

## 세 줄 요약

- IB 외에도 코드로 오토 레이아웃을 잡아주는 constraint 지정을 할 수 있다.
- 값을 지정할 때 양수, 음수 값이 모두 들어갈 수 있음에 유의하자
- 얼른 translatesAutoresizingMaskIntoConstraints에 대해서도 글을 쓰자..!
