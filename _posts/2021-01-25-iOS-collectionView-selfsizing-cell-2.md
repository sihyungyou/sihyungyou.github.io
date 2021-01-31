---
layout: post
title: "iOS) CollectionView Self-Sizing Cell 2탄"
tags: [iOS, Swift]
comments: true
---

> 컬렉션뷰의 셀 크기를 동적으로 조절해보자  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

테이블뷰의 셀 재사용 과정을 공부하며 글을 쓰다가 문득 self-sizing cell 방법에 대해 다시 이해하고 도전해보고 싶어졌다. 그래서 이런저런 글을 읽다가 정말 좋은 글을 발견해 해당 방법으로 직접 실습을 하고 포스팅해보려한다.

아래 코드와 같이 각 cell의 height과 color를 미리 데이터소스로 사용했을 때의 결과를 첨부했다.

```swift
var heights: [Int] = [60, 200, 150, 30, 80, 170, 200]
var colors: [UIColor] = [.systemRed, .systemIndigo, .systemBlue, .systemTeal, .systemYellow, .cyan, .brown]
```

```swift
extension ViewController: UICollectionViewDelegate, UICollectionViewDataSource {

    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return heights.count
    }

    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: SelfSizingCell.identifier, for: indexPath) as? SelfSizingCell else {
            return UICollectionViewCell()
        }

        let height = heights[indexPath.row]
        let color = colors[indexPath.row]

        cell.configureHeight(with: height)
        cell.configureColor(with: color)

        return cell
    }

}
```
![1](https://user-images.githubusercontent.com/35067611/105681801-f6e3e280-5f34-11eb-8a12-519188629e7a.png)

컬렉션뷰의 delegate, datasource, register 등 기본적인 준비는 평소에 컬렉션뷰를 사용하던대로 동일하게 했다고 가정하고 이에 추가로 거쳐야할 스텝들에 대해서 설명해보겠다. 또한 (개인적으로) 평소에도 컬렉션뷰나 테이블뷰의 셀을 xib 파일로 만드는 것을 선호하는데 이번에도 xib 파일로 셀을 구성한다고 가정하겠다.

## Step 1. MainView

가장 먼저 xib 파일 셀에 모든 뷰 요소들을 포함할 MainView를 생성하고 셀 자체에 top, bottom, leading, trailing constraint를 0으로 주었다.

![2](https://user-images.githubusercontent.com/35067611/105681815-fba89680-5f34-11eb-82ed-4655672ad661.png)

이 MainView 하위에 SubView를 넣고(이번 예제에서는 색깔을 갖는 하나의 Subview지만 레이블이나 이미지 등 여러 서브뷰들을 포함시킬 수 있다) 마찬가지로 SubView의 top, bottom, leading, trailing constraint를 MainView에 대해 0으로 주었다. 이렇게 최상단에 MainView를 하나 더 두는 것이 핵심인데, SubView의 크기에 따라 MainView가 변하도록 하고 이 MainView의 constraint를 코드로 조정하여 width를 고정시키면 height이 동적으로 변한다.

## Step 2. UICollectionViewFlowLayout

다시 컬렉션뷰를 host하는 ViewController로 돌아와보자. delegate, dataSource, Cell 등록을 마치고 컬렉션뷰의 estimatedItemSize에 `UICollectionViewFlowLayout.automaticSize`을 넣어준다.

```swift
collectionView.delegate = self
collectionView.dataSource = self

collectionView.register(UINib(nibName: SelfSizingCell.identifier, bundle: nil), forCellWithReuseIdentifier: SelfSizingCell.identifier)

// 여기가 핵심!
if let collectionViewLayout = collectionView.collectionViewLayout as? UICollectionViewFlowLayout {
    collectionViewLayout.estimatedItemSize = UICollectionViewFlowLayout.automaticSize
}
```

여기서 estimatedItemSize와 UICollectionViewFlowLayout, automaticSize의 역할을 알고 넘어가보자.

`estimatedItemSize`는 이름 그대로 "The estimated size of cells in the collection view." 이다. 이 프로퍼티를 지정해준다고 해서 지정된 값으로 컬렉션뷰 셀의 크기가 고정되는 것은 아니지만 동적으로 크기를 바꾸어야 할 때 실제 크기와 estimatedItemSize의 값의 차이가 적을수록 계산이 빨라 퍼포먼스 향상에 도움이 된다고 한다.

`UICollectionViewFlowLayout`는 공식문서에 "A layout object that organizes items into a grid with optional header and footer views for each section."라고 설명되어있다. header, footer가 있다면 각 섹션의, 없다면 전체 셀들을 grid하게 organize하는 레이아웃 객체이다. 이 객체의 프로퍼티 중 automaticSize가 있다.

`automaticSize`는 self-sizing cell을 위한 placeholder size이다. 이 프로퍼티의 Discussion을 읽어보자

Set this constant as the value for the `estimatedItemSize` property to enable self-sizing cells for your collection view. This is a non-zero, placeholder value that tells the collection view to query each cell for its actual size using the cell’s `preferredLayoutAttributesFitting(_:)` method.

컬렉션뷰의 셀이 self-sizing 되도록 하기 위해서는 이 값을 estimatedItemSize에 대입하라고 설명한다. 그러면 **preferredLayoutAttributesFitting** 메소드를 활용해 셀의 실제 사이즈를 불러오도록 할 수 있다고 한다. 이 메소드는 [이전 포스팅](https://sihyungyou.github.io/iOS-collectionView-selfsizing-cell/)에서 다룬 적이 있는데 역시 self-sizing cell에 대한 이야기이다. 결국 기존에 포스팅한 방법을 좀 더 코드 레벨에서 쉽게 구현해주는 방법인 것이다!

## Step 3. Width 고정

이제 마지막 단계이다. cell의 height를 동적으로 변경하고자한다면 MainView의 width를 고정시켜주면 된다. 물론 반대의 경우에는 height를 고정시키면 된다. 아래 코드와 위의 최종결과물을 함께 보면 직관적으로 이해가 될 것이다.

```swift
class SelfSizingCell: UICollectionViewCell {

    static let identifier = "SelfSizingCell"

    @IBOutlet weak var mainView: UIView!
    @IBOutlet weak var subView: UIView!

    override func awakeFromNib() {
        super.awakeFromNib()

        self.layer.cornerRadius = 5

        // width 고정
        mainView.widthAnchor.constraint(equalToConstant: UIScreen.main.bounds.size.width - 40).isActive = true
    }

    func configureHeight(with height: Int) {
        subView.heightAnchor.constraint(equalToConstant: CGFloat(height)).isActive = true
    }

    func configureColor(with color: UIColor) {
        subView.backgroundColor = color
    }

}
```

## 세 줄 요약

- estimatedItemSize에 automaticSize를 대입함으로써 self-sizing이 가능하다.
- 이번에 발견한 self-sizing 방법은 결국 기존 포스팅에서 다루었던 방법을 다른 코드레벨에서 구현한 것이다 (이렇게 연결되다니!)
- width를 동적으로 바꾸려면 height를, 반대의 경우엔 width를 고정된 constraint로 주면된다.

## References

[Self-sizing UICollectionViewCell: the quickest and simplest way](https://medium.com/@giusecapo/self-sizing-uicollectionviewcell-the-quickest-and-simplest-way-94e8d1c89594)

[애플공식문서 - UICollectionViewFlowLayout](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout)

[애플공식문서 - automaticSize](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout/1779556-automaticsize)
