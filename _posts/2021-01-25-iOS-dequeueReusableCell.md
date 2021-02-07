---
layout: post
title: "iOS) dequeueReusableCellWithIdentifier 과정과 사용이유"
tags: [iOS, Swift]
comments: true
---

> 테이블뷰, 컬렉션뷰의 셀 재사용에 대해 생각해보자  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

부스트포켓 프로젝트를 진행하다가 아래와 같은 버그를 경험한 적이 있다. 오늘은 이 버그를 해결할 수 있었던 방법과 해결과정에서 추가적으로 공부할 수 있었던 `dequeueReusableCellWithIdentifier` 과정에 대한 이해와 사용 이유를 정리해보았다.

## 현상
![1](https://user-images.githubusercontent.com/35067611/105666246-30f4ba80-5f1c-11eb-9d91-a2124efc6afb.gif)

위 gif를 자세히 보면 하나의 셀을 선택해서 체크박스를 표시한 후 스크롤을 내리면 선택하지 않은 셀들에도 체크박스가 표시된 것을 확인할 수 있다. 이런 현상은 테이블뷰의 `dequeueReusableCell` 때문이다. 테이블뷰나 컬렉션뷰는 모두 셀을 "Reuse", 재사용하는데 이런 셀의 재사용 구조가 큐로 되어있어서 "dequeue"라는 키워드도 등장하는 것이다. 아래 사진을 보면 이해가 더 쉽게 된다.

![2](https://user-images.githubusercontent.com/35067611/105666256-37833200-5f1c-11eb-8493-4d5549e7236f.png)

테이블뷰에서 위로 스크롤되어 화면에서 사라지는 셀들은 큐로 들어가고 큐의 front에 있는 셀이 화면의 하단에서 올라와 화면에 보여지는 셀로 사용된다. 이렇게 dequeue, reuse 되는 과정이 흔히 사용했던 `cellForRowAt` delegate 메소드에서 `dequeuReusableCellWithIdentifier`함수를 통해 이루어진 것이다. 그래서 셀에 configure되는 데이터소스의 내용은 다르지만 셀 자체는 재사용되기 때문에 체크박스가 사라지지 않고 계속 보이는 상태로 있게된다.

## 해결방법

결국 이 현상은 셀의 속성을 초기화시켜주지 않은 채 재사용하기 때문에 발생한다. 그래서 해결방법도 간단하다. 재사용되는 셀의 속성을 초기화시켜주면 된다.

```swift
// tableViewCell.swift
override func prepareForReuse() {
    super.prepareForReuse()

    self.accessoryType = .none
}
```

테이블뷰 셀은 `prepareForReuse` 함수를 제공하는데 여기서 셀의 뷰 측면의 속성들을 초기화시켜주면 된다. 아래 다이어그램을 보면 위에서 언급한 `cellForRowAt` 함수(셀의 재사용)와 `prepareForReuse` 함수(셀의 속성 초기화)가 불리는 시점을 명확히 설명해주고 있다.

![3](https://user-images.githubusercontent.com/35067611/105666257-381bc880-5f1c-11eb-852f-024dbc043269.png)

여기서 한 가지 더 고려해야할 것이 있다. 이렇게 모든 재사용되는 셀에 대해 체크박스를 해제해주면 실제로 선택된 셀도 체크박스가 보이지 않게되므로, 데이터소스를 확인해서 선택된 셀은 체크박스를 다시 보이도록 설정해줘야한다.

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    guard let cell = tableView.dequeueReusableCell(withIdentifier: CountryTableViewCell.identifier, for: indexPath) as? CountryTableViewCell else { return UITableViewCell() }

    let currentCountry = countries[indexPath.row]

    if currentCountry.isSelected {
        cell.accessoryType = .checkmark
    }

    cell.configure(with: currentCountry)

    return cell
}
```

## 굳이 왜 DequeueReusableCellWithIdentifier를 사용할까?

이쯤되면 이런 생각이 든다. "애초에 dequeue 하는 방식이 아니라 실제 데이터소스 개수 만큼 cell 객체를 생성해서 테이블뷰에 보여줬으면 되는 거 아닌가?" 물론 가능하다. 예를 들면 아래와 같이 코드를 작성하는 방법이 있다. 테이블뷰에 보여줘야 할 데이터가 1000개라고 가정한 경우이다.

```swift
import UIKit

class BadTableViewController: UIViewController {

    @IBOutlet weak var tableView: UITableView!

    var cells: [UITableViewCell] = []

    override func viewDidLoad() {
        super.viewDidLoad()

        cellsSetup()
        tableViewSetup()
    }

}

extension BadTableViewController {

    func cellsSetup() {
        for _ in 0...1000 {
            // 여기서 1000개의 Cell 인스턴스를 직접 생성하고 append하는 것에 주목하자
            let cell = UITableViewCell()
            cell.textLabel?.text = "Wow so bad"
            cells.append(cell)
        }
    }

    func tableViewSetup() {
        tableView.rowHeight = UITableViewAutomaticDimension
        tableView.estimatedRowHeight = 350
    }

}

extension BadTableViewController: UITableViewDataSource {

    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return cells.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {

        // 위에서 생성해놓은 1000개의 cell 인스턴스 배열에서 꺼내쓰기만 하면 된다.
        // 즉, dequeu 과정이 없다!
        let cell = cells[indexPath.row]
        return cell
    }

}
```

이런 방식의 테이블뷰 사용은 데이터 소스와 테이블뷰 셀 인스턴스가 1:1로 대응되어 위에서 설명한 체크마크가 중복되는 등의 문제도 없을 것이다. 하지만 데이터가 1000개가 아니라 10만개라고 생각해보자. 이런 방식으로 cell을 모두 생성한다면 메모리 사용량 측면에서 효율이 매우 나쁜 설계일 것이다. 그래서 애플은 `dequeueReusableCellWithIdentifier`방식으로 테이블뷰, 컬렉션뷰를 사용하도록 권장하고 실제로 훨씬 좋은 방법인것이다.

## 개선점을 찾아보자

이렇게 dequeue하는 방식으로 메모리 사용량을 줄이는 방법이 일반적인 상황에서는 충분하다. 하지만 여기서 조금 더 개선해볼 수 없을까? 예를 들어 셀에 이미지가 포함되는 경우를 생각해보자. 아마 실제 상용되는 앱에서도 가끔 경험했을 수 있는데, 인터넷 상황이 좋지 않은 상황에서(이미지 로드에 시간이 더 걸리는 상황에서) 너무 빠르게 스크롤을 하다보면 이미지가 있어야할 곳이 아닌 다른 셀에서 보여지는 경우가 종종있다. 이것은 셀이 재사용되기 때문인데 텍스트와 이미지가 rolled over(뭐라고 번역해야할지..) 되어 올바른 셀에 위치하지 않는 것이다.
![4](https://user-images.githubusercontent.com/35067611/105666258-381bc880-5f1c-11eb-94f0-a6df3b33de7d.gif)  

이런 현상 역시 `prepareForReuse()` 메소드를 통해 해결(개선)할 수 있다.

```swift
override func prepareForReuse() {
    mainImageView.af_cancelImageRequest() // NOTE: - Using AlamofireImage
    mainImageView.image = nil
}
```

위 코드처럼 셀이 dequeue되기 직전 이미지 요청이 있다면 취소하고, imageView의 image를 nil로 바꾸는 것만으로도 이미지가 로드되지 않았는데 엉뚱한 이미지가 보여지는 현상을 고칠 수 있다.
![5](https://user-images.githubusercontent.com/35067611/105666266-3b16b900-5f1c-11eb-8d95-3c909b32dbbb.gif)

## 세 줄 요약

- 사실 dequeue 없이도 테이블뷰에 데이터를 보여줄 수는 있다.
- 하지만 dequeue 과정은 메모리 효율성 측면에서 꼭 필요하다.
- 셀이 재사용되는 흐름을 잘 이해하여 `prepareForReuse()` 메소드와 `dequeue` 메소드에서 적절히 코드를 작성하면 더 좋은 사용성을 제공할 수 있다.

## References

[Solving duplicated / repeating cells in Table view](https://fluffy.es/solve-duplicated-cells/)

[Why we use dequeueReusableCellWithIdentifier](https://medium.com/ios-seminar/why-we-use-dequeuereusablecellwithidentifier-ce7fd97cde8e)
