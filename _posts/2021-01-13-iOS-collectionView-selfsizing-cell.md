---
layout: post
title: "iOS) CollectionView Self-Sizing Cell"
tags: [iOS, Swift]
comments: true
---

> 동적인 컬렉션뷰 셀 사이즈  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

CollectionView를 사용하다보면 cell의 size가 동적으로 계산되는 경우를 반드시 만나게 된다. 유저가 입력한 만큼의 UILabel을 보여줘야할 수도 있고 cell에 들어가는 요소가 시간에 따라 바뀌는 상황도 자주 있기 때문이다. 이걸 이해하는데 정말 애먹었는데 물론 아직 완-벽하지는 않지만 그래도 어떤 요소들을 신경써야 하는지 기록해두려 한다.

## 1. UIFlowLayout 적용하기 (ViewController)

먼저 코드를 살펴보자. 아래 코드는 단순히 현재 view의 전체 width / height을 cell에 적용시킨다. 여기서 주목할 점은 실제로 이 크기의 cell이 생성되는 것이 아니라 `estimated`, 즉 미리 틀을 잡아주기만 한다는 것이다.

```swift
private func setupCollectionViewLayout() {
    let flowLayout = UICollectionViewFlowLayout()

    let width = self.view.frame.width
    let height = self.view.frame.height
    flowLayout.estimatedItemSize = CGSize(width: width, height: height)

    collectionView.collectionViewLayout = flowLayout
}
```

미리 틀을 잡을 뿐 width, height 둘 다 얼마든지 동적으로 변경될 수 있으므로 이 수치에는 크게 신경 쓸 필요가 없다.

## 2. preferredLayoutAttributesFitting 함수 이용하기 (Cell.swift)

```swift
override func preferredLayoutAttributesFitting(_ layoutAttributes: UICollectionViewLayoutAttributes) -> UICollectionViewLayoutAttributes {
    super.preferredLayoutAttributesFitting(layoutAttributes)
    layoutIfNeeded()

    let size = contentView.systemLayoutSizeFitting(layoutAttributes.size)

    var frame = layoutAttributes.frame
    frame.size.height = ceil(size.height)

    layoutAttributes.frame = frame
    return layoutAttributes
}
```

`preferredLayoutAttributesFitting` 함수는 UICollectionViewCell에 포함되어있으며 역할은 "Gives the cell a chance to modify the attributes provided by the layout object.", layout object로부터 주어진 크기에 대해 한번 조정할 기회를 주는 것이다.

위 함수를 내가 원하는대로 재정의한 것인데 width는 flowLayout이 준 값으로 고정시키고 height만 동적으로 바꾸고 싶어서 size.heigt만 재조정을 하는 로직이다. 이 때 `systemLayoutSizeFitting` 함수는 "Returns the optimal size of the view based on its current constraints." 현재 constraint를 준수하면서 가장 optimal한 크기를 반환해준다.

그러니까 컬렉션뷰의 아이템에 적용되는 레이아웃 객체인 `UICollectionViewLayoutAttributes`의 frame을 내가 재정의한 frame 값으로 바꿔치는 것이고, 위 함수에서 바꿔치는 값은 optimal 한 크기라는 것이다.

아래와 같은 구조로 하나의 버튼, 하나의 레이블을 갖는 컬렉션 뷰 셀에 위 함수들을 적용시켜보자. 버튼은 왼쪽에서 10만큼, 버튼과 레이블도 10만큼, 레이블은 오른쪽 벽에서 10만큼 떨어져있다.

![1](https://user-images.githubusercontent.com/35067611/104598784-54b53680-56ba-11eb-8fb2-70303ea046b2.png)

결과는 아래와 같다. 크기를 제대로 보기 위해 배경색상을 입혔다. height는 정상적으로 잘 적용이 된 것 같다! (참고로 레이블의 Lines = 0, Line braek mode는 word wrap으로 설정해줘야 여러 줄의 text를 나타낼 수 있다)

![2](https://user-images.githubusercontent.com/35067611/104598796-567efa00-56ba-11eb-88d0-774908f59a01.png)

하지만 버튼의 위치가 엉망진창이다. 사실 정확히 말하면 버튼 위치에는 문제가 없고 width가 늘어나고 줄어들고 있다. 먼저 Content Hugging, Compression Resistance의 개념을 공부하고 다시 이 사진으로 돌아오자.

## 3. Content Hugging vs. Compression Resistance

`Content Hugging Priority`는 스스로를 꽉 안고서 (hugging) 커지기를 거부하는 것이다. Content Hugging Priority가 높다면 자신보다 우선순위가 낮은 것들보다 버티고 버티다가 (저항하다가) 커진다는 뜻이다. 다른 아이템들이 모두 커진 후에야 거기에 맞춰 크기가 조정된다. → **커져야 하는 것에 대한 저항**

반대로 `Compression Resistance Priority`는 작아지기를 거부하는 것이다. 이 우선순위가 높을수록 최대한 저항하다가 나중에 작아지게 된다. 주변의 다른 아이템들의 최대한 축소된 이후에야 영향을 받기 시작한다. → **작아져야 하는 것에 대한 저항**

다시 초기 사진으로 돌아와보자.

![3](https://user-images.githubusercontent.com/35067611/104598813-5979ea80-56ba-11eb-9885-51b0f8119b66.png)

현재 버튼과 레이블의 Content Hugging Priority는 각각 250, 251로 만약 커져야하는 상황이 온다면 둘 중에 버튼의 width가 커질 것이다. 그래서 위 버튼이 커진 모습을 볼 수 있다. 하지만 빨강버튼은 오히려 찌그러들었다!

이는 Compression Resistance 때문인데 두 아이템 모두 크기가 Cell에 들어가지 못할정도로 커서 누군가가 줄어들어야하는 상황이다. 현재 둘의 Compression Resistance는 모두 750으로 동일한데, Content Hugging Priority는 버튼이 더 우선순위가 높으므로 레이블이 커지는(버튼이 작아지는) 것 같다.

그렇다면 레이블이 커지되, 버튼의 intrinsic size를 침범하지 않을 정도로만 커지게 하기 위해서는 어떻게 해야할까? 레이블이 점점 커지다가 버튼의 크기를 줄어들게 하려 할 때 Compression Resistance Priority를 비교해서 버튼이 intrinsic size 보다 더 작아지지 않도록 저항하게 만들면 된다.

즉, 정리하면 레이블과 버튼이 Cell의 사이즈에 여유공간이 많이 남아서 둘 중 하나는 커져야한다면 레이블을 크게 만들되, 버튼의 intrinsic size보다는 커지지 못하게 우선순위를 정해주면 된다.

1. Content Hugging Priority : **버튼 251, 레이블 250**
2. Compression Resistance Priority : **버튼 751, 레이블 750**

![4](https://user-images.githubusercontent.com/35067611/104598816-5aab1780-56ba-11eb-841b-0e8f0c7a4eb0.png)

![5](https://user-images.githubusercontent.com/35067611/104598819-5bdc4480-56ba-11eb-9d00-43c4935d8be9.png)

아주 예쁘게 결과가 나왔다! (+ 배경색상 없앤 버전)

## References

[Content hugging vs Compression resistance 차이점 알기!](https://ontheswift.tistory.com/21)

[[iOS Swift] Content Hugging, Compression Resistance Priority](https://m.blog.naver.com/PostView.nhn?blogId=jdub7138&logNo=220963551062&proxyReferer=https:%2F%2Fwww.google.com%2F)
