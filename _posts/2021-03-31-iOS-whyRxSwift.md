---
layout: post
title: "Why RxSwift?"
tags: [iOS, Swift]
comments: true
---

> RxSwift는 왜 이렇게 핫할까?  

iOS 개발자들 사이에서 RxSwift가 상당히 핫하다. 2020 Let's Swift 컨퍼런스에서 자주 쓰는 오픈소스에 대한 투표(?)를 했는데 아래 사진에서 볼 수 있듯 RxSwift가 압도적이었다. 물론 표본이 적은 편이지만 그래도 현업 iOS 개발자들이 컨퍼런스에 모여 직접 투표했으니 의미가 있다고 생각한다.

<img width="942" alt="1" src="https://user-images.githubusercontent.com/35067611/113367876-31845400-9398-11eb-9f25-19b156ce754a.png">

사진출처 : [레츠스위프트 2020 테크토크 Day#1 영상](https://youtu.be/MMyfZ3pX2TA)

그렇다면 RxSwift는 무엇이며 왜 이렇게 사랑받고 있는지 그 이유에 대해서 몇가지 예제를 직접 작성해보면서 알아보자.

# Rx란?

먼저 `Rx`가 무엇인지 알아야한다. RxSwift는 하나의 새로운 언어가 아니라 Rx가 지원하는 여러 언어 중 하나이기 때문이다. (RxJava, RxScala 등 여러 언어에서 Rx가 지원된다)

Rx는 `Reactive Extensions`을 사용하는 라이브러리이다. 즉, Reactive Programming을 쉽게 할 수 있도록 돕는 역할을 한다. `Reactive Programming`은 데이터의 흐름과 그에 대한 처리를 정의해놓고, 흐름에서 변경사항이 생기면 미리 정의해둔 방식에 따라 변화를 주는 프로그래밍 방식이다. 결국 이름부터 반응형인데 변화에 실시간으로 반응하기 때문인 것 같다.

여기서 중요한 포인트는 Reactive Programming은 하나의 패러다임일 뿐이기에 Rx를 사용하지 않아도 reactive하게 프로그램을 구현할 수 있다. 다만, Rx는 Extension 이라는 단어에서 알 수 있듯, 이를 더 편하고 간단하게 구현할 수 있도록 돕는 도구이다.

# Why RxSwift?

이번 포스팅에서는 RxSwift의 문법적인 사용법 등 보다는 어떤 필요에 의해 RxSwift가 여러 프로젝트에 도입되는지 그 이유, **WHY**에만 집중하겠다. 이미 RxSwift를 사용하기로 결정했다면 이 글은 스킵하길 추천한다(?)

## #간결함

여기서 이야기할 간결함이라는 단어는 코드의 길이, 가독성, 이해도보다는 `비동기적 작업의 동기화`라는 측면에서의 간결함이다. 개발을 하면서 특정 작업을 비동기적으로 처리해야하는 상황을 종종 마주하게된다. 아주 간단한 예로 JSON 데이터를 다운로드 받고 그 결과를 화면에 보여주는 프로그램을 생각해보자.

<img src = "https://user-images.githubusercontent.com/35067611/113367889-36490800-9398-11eb-8ceb-36403d787349.gif" width="350px">

예제 출처 : [곰튀김님 RxSwift 4시간만에 끝내기 시즌2](https://youtu.be/iHKBNYMWd5I)

로드 버튼을 누르면 JSON 데이터를 요청하고 결과를 반환받으면 UI를 업데이트한다. 당연히 데이터 요청 시 UI가 freeze 되면 안되므로 비동기로 처리해주어야한다. 아래와 같이 코드를 짤 수 있을 것이다.

```swift
@IBAction func onLoad(_ sender: Any) {
    // step 1 : onLoad에서 비동기 작업 처리
    DispatchQueue.global().async {
        let url = URL(string: URLSTRING)!
        let data = try! Data(contentsOf: url)
        let json = String(data: data, encoding: .utf8)

        DispatchQueue.main.async { [weak self] in
            self?.textView.text = json
        }
    }
}
```

코드가 좀 지저분하다. onLoad 함수에서는 다운로드하라고 지시만 하고 비동기 작업을 함수로 분리해보자.

```swift
@IBAction func onLoad(_ sender: Any) {
    // step 2
    downloadJSON() { [weak self] json in
        self?.textView.text = json
    }
}

// step 2 : onLoad 함수 정리를 위해 비동기 작업 함수로 분리
private func downloadJSON(completion: @escaping (String?) -> Void) {
    DispatchQueue.global().async {
        let url = URL(string: URLSTRING)!
        let data = try! Data(contentsOf: url)
        let json = String(data: data, encoding: .utf8)

        DispatchQueue.main.async {
            completion(json)
        }
    }
}
```

여기서부터 문제가 시작된다. DispatchQueue 클로저 내부에서 json 데이터를 return 할 수 없기 때문에 escaping 클로저를 통해서 비동기 작업의 결과를 반환해주어야한다는 것이다. 이렇게 되면 비동기로 처리되는 작업들이 이전의 데이터에 의존적으로 엮여서 실행되는 경우에 상당한 callback depth(흔히 말하는 콜백지옥)이 생긴다. 그리고 onLoad 함수가 구현된 모습에서 알 수 있듯 코드를 작성하거나 읽을 때 "비동기적으로" 생각해야한다는 것이다. 우리는 동기적으로 생각하는데 익숙하고 그런 생각의 흐름이 코드로 나타났을 때 훨씬 직관적이다. 그렇다면 어떻게 비동기 작업을 "동기적"인 코드로 작성할 수 있을까?

```swift
@IBAction func onLoad(_ sender: Any) {
    // step 3 : 비동기의 결과를 return 값으로 (동기적으로) 받기
    let json: JSONData<String?> = downloadJSON(with: URLSTRING)

    json.whenArrived { [weak self] json in
        self?.textView.text = json
    }
}
```

```swift
// step 3
class JSONData<T> {
    private let task: (@escaping (T) -> Void) -> Void

    init(task: @escaping (@escaping (T) -> Void) -> Void) {
        self.task = task
    }

    func whenArrived(task: @escaping (T) -> Void) {
        self.task(task)
    }
}

// step 3
private func downloadJSON(with url: String) -> JSONData<String?> {
    return JSONData { task in
        DispatchQueue.global().async {
            let url = URL(string: url)!
            let data = try! Data(contentsOf: url)
            let json = String(data: data, encoding: .utf8)

            DispatchQueue.main.async {
                task(json)
            }
        }
    }
}
```

첫번째 코드블럭을 보면 step 2와 확연히 차이가 난다. json 이라는 상수에 데이터 요청의 결과 (비동기 작업의 결과)를 동기적으로 반환받아 대입하고 있다! 바로 이 포인트가 Rx의 첫번째 필요성이다. (아직 RxSwift를 적용한 코드는 아니지만 RxSwift를 도입하고자 하는 이유에 하나씩 다가가고 있는 중이다) 실제로 [RxSwift의 공식 깃헙](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/Why.md#benefits)에서 설명하는 RxSwift의 이점 중에 "Understandable and concise <- Raising the level of abstraction and removing transient states" 내용이 포함되어있다.

두번째 코드블럭은 비동기 작업을 수행하는 클래스를 정의하고 그 클래스의 인스턴스를 동기적으로 반환해줄 수 있도록 구현한 것이다. *코드는 곰튀김님의 영상을 참고했다. 그렇다면 이제 **RxSwift가 정말 이런 이점을 가져다주는지** 코드에 적용해보자.

```swift
import RxSwift

@IBAction func onLoad(_ sender: Any) {
    // step 4 : 실제 RxSwift 적용
    let json = URLSession.shared.rx
        .data(request: URLRequest(url: URL(string: URLSTRING)!))
        .map { String(data: $0, encoding: .utf8) }

    json.bind(to: textView.rx.text)
}
```

(일단 RxSwift 문법에는 신경쓰지 말고 느낌만 보자) URLSession을 사용하여 JSON 데이터를 처리하는 비동기 작업의 결과가 json이라는 상수에 동기적으로 대입되었다. 그리고 json 데이터는 화면의 textView에 바인딩되어 상태가 변할 때마다 UI 업데이트까지 이루어진다.

정리하면 RxSwift는 비동기 작업을 처리하는 데 코드를 간결하게 쓰기 위해 도입한다. 위에서 언급했듯 "간결하다"는 단어의 의미를 두 가지 측면으로 생각할 수 있는데 말 그대로 더 적은 양의 가독성 높은 코드, 혹은 `async`를 `sync`하게 작성하는 코드라고 할 수 있다. 우리는 sync하게 생각하는 데 익숙하고 그런 생각의 흐름이 훨씬 직관적이기 때문이다.

이번 예제에서는 코드의 양이 드라마틱하게 줄거나 가독성이 딱히 높아졌다는 생각이 들지 않을 수 있지만 이는 매우 간단한 예제이기 때문에 그렇다. 비동기 작업들이 복잡하게 얽히기 시작하면 RxSwift는 더 강한 힘을 발휘하게된다.

## #상태 #관찰 #리액티브프로그래밍

이번엔 상태, 관찰, 그리고 리액티브 프로그래밍 패러다임이라는 관점에서 RxSwift를 살펴보자.

<img src = "https://user-images.githubusercontent.com/35067611/113367890-377a3500-9398-11eb-9b79-56d3f6d2310c.gif" width="350px">

면접에서 비동기적으로 작동하는 태스크 A, B, C가 있는데 그것들이 모두 끝났는지 어떻게 알 수 있냐는 질문을 받은 적이 있다. 답변을 횡설수설했는데 각 비동기 태스크의 완료여부를 표시하는 flag 변수를 하나 두고 무한루프에서 이 변수를 관찰(Rx적으로 말하자면 subscribe)하는 방법이 가능할 것 같다. 혹은 DispatchGroup을 사용하는 방법도 떠오른다. (직접 코드로 짜보질 않아서 확신은 없다)

면접에서 질문받은 경우가 정확히 이번 예제이다. Task1, 2, 3은 모두 다른 속도로 작업이 수행된다. 각각의 작업은 완료되면 불렛을 초록색으로 바꾸고 세 작업이 모두 완료되면 프로그레스 바를 초록색으로 바꾼다.

여기서 상태와 관찰에 주목해보자. 버튼에 tap이 발생함으로써 `상태`가 변한다. 이 상태변화에 따라 progress가 시작된다. 이 progress 역시 Task1, 2, 3 각각의 속도에 따라 진행척도라는 상태가 변한다. 이 상태변화는 UI에 진행정도를 나타내는 막대의 widthConstraint에 영향을 주고 완료됐을 때 불렛의 색상도 변경시킨다. 이렇게 여러 상태들이 비동기적으로 변할 때 이것들을 `관찰`하고 있다가 어떤 조건이 만족되면 특정 작업을 수행해야한다.

RxSwift의 문법이 익숙치 않다면 코드를 읽는 것이 쉽지 않겠지만 이번에도 느낌만 보자..!

```swift
private func bindButtonTaps() {
    task1Button.rx.tap
        .flatMapLatest { Observable<Int>.interval(.milliseconds(250), scheduler: ConcurrentDispatchQueueScheduler.init(qos: .default)) }
        .map { $0 * 30 }
        .bind(to: task1Progress)
        .disposed(by: disposeBag)

    task2Button.rx.tap
        .flatMapLatest { Observable<Int>.interval(.milliseconds(150), scheduler: ConcurrentDispatchQueueScheduler.init(qos: .default)) }
        .map { $0 * 30 }
        .bind(to: task2Progress)
        .disposed(by: disposeBag)

    task3Button.rx.tap
        .flatMapLatest { Observable<Int>.interval(.milliseconds(50), scheduler: ConcurrentDispatchQueueScheduler.init(qos: .default)) }
        .map { $0 * 30 }
        .bind(to: task3Progress)
        .disposed(by: disposeBag)
}
```

각 버튼에 탭 이벤트가 발생했을 때 각기 다른 interval의 타이머를 발생시켜 progress에 바인딩하고 있다. 이 때 task1Progress, task2Progress 등은 각 작업의 진행정도를 저장하는 변수라고 생각하자. 이 변수들은 버튼에 tap이 발생하는지 `상태를 관찰`하고 있다.

```swift
private func bindTaskComplete() {
    Observable.combineLatest(task1Progress, task2Progress, task3Progress, resultSelector: { (s1: Int, s2: Int, s3: Int) in s1 == 300 && s2 == 300 && s3 == 300 })
        .observe(on: MainScheduler.instance)
        .subscribe(onNext: { [weak self] result in
            if result {
                self?.task1ProgressBar.backgroundColor = .systemGreen
                self?.task2ProgressBar.backgroundColor = .systemGreen
                self?.task3ProgressBar.backgroundColor = .systemGreen
            }
        })
        .disposed(by: disposeBag)
}
```

task1, 2, 3의 진행정도를 저장하는 task1Progress, task2Progress, task3Progress 변수를 관찰하고 있다가 세 변수 모두 진행이 완료되었다면 모든 진행막대의 색깔을 바꾸는 코드이다.

코드를 보면 각기 다른 소스로부터의 상태변화를 관찰하고 그 관찰결과를 쉽게 조합하여 조건에 맞는 작업을 구현했다. 하지만 RxSwift 없이 이런 작업을 수행하려면 flag를 여러개 두고 무한루프를 돌면서 계속 그 flag 값을 확인하는 등 부자연스럽게 코드를 작성해야 할 것 같다. (RxSwift를 사용하지 않되 더 나은 방법이 뭐가 있을지 현재로썬 잘 모르겠다)

## #RxMVVM

이번엔 RxSwift가 MVVM 아키텍처 패턴과 쌍을 이룰 때 어떤 이점을 갖는지 알아보자.

<img src = "https://user-images.githubusercontent.com/35067611/113367891-3812cb80-9398-11eb-9084-7471facdb31f.gif" width="350px">

예제 출처 : [곰튀김님 RxSwift 4시간만에 끝내기 시즌2](https://youtu.be/iHKBNYMWd5I)

장바구니 예제는 본격적으로 RxSwift와 MVVM을 접목시켜보았다. Rx 자체에 익숙해지는 것도 쉽진 않았지만 MVVM이라는 아키텍처 패턴에 적용하는 것도 매우 낯설었다. 핵심은 View를 최대한 멍청하게 만드는 것이다. 예를 들어 TableViewCell에 데이터를 채우는(configure하는) 과정에서 최대한 추가적인 연산을 지양하여 뷰는 그저 뷰모델로부터 전달받는 데이터를 화면에 뿌리는 역할에만 충실하도록 하는 것이다.

```swift
func configure(menu: MenuItem, count: Int) {
      menuNameLabel.text = menu.name
      menuPriceLabel.text = "\(menu.price)원"
      menuCountLabel.text = "(\(count)개)"
  }
```

뷰는 모델에 대해 전혀 알지 못한다. 즉 의존도가 전혀 없는데 뷰 → 뷰모델 → 모델 단방향적으로 의존도가 흐르게된다. 뷰는 **뷰모델의 상태변화에 따라 UI를 업데이트할 뿐**이다. 그리고 뷰모델의 상태와 뷰를 RxSwift를 통해 바인딩해주었다.

```swift
// ViewModel 정의되어 있는 menus, price, count 변수
var menus = BehaviorRelay<[(menu: MenuItem, count: Int)]>(value: [])
lazy var totalPrice = menus.map { $0.map{ $0.menu.price * $0.count }.reduce(0, +) }
lazy var totalCount = menus.map { $0.map { $0.count }.reduce(0, +) }
```

```swift
// ViewController에서 UI와 뷰모델 내부의 변수(상태) 바인딩

// bind totalPriceLabel
menuItemListViewModel.totalPrice
    .map { $0.currencyKR() }
    .bind(to: totalPriceLabel.rx.text)
    .disposed(by: disposeBag)

// bind totalCountLabel
menuItemListViewModel.totalCount
    .map({ "총합 \($0)개" })
    .bind(to: totalCountLabel.rx.text)
    .disposed(by: disposeBag)

// bind tableView
menuItemListViewModel.menus
    .bind(to: tableView.rx.items(cellIdentifier: MenuCell.identifier, cellType: MenuCell.self)) { index, element, cell in
        cell.selectionStyle = .none
        cell.configure(menu: element.menu, count: element.count)

        cell.itemOnChanged = { [weak self] inc in
            let newCount = max(0, element.count + inc)
            self?.menuItemListViewModel.changeCount(at: index, to: newCount)
        }

    }.disposed(by: disposeBag)
```

**RxSwift를 사용하지 않는다면** 아래와 같이 뷰모델에서 didSet/willSet을 정의해주는 방법이 있다.

```swift
// travelListViewModel에 정의된 travels, 즉 뷰모델 내부 "상태"
var travels: [TravelItemViewModel] = [] {
    willSet {
        DispatchQueue.main.async { [weak self] in
            self?.didFetch?(newValue)
        }
    }
}
```

코드의 복잡성에서 크게 차이가 나지는 않지만 코드를 들여다보면 상태가 변할 때 정의되어야 하는 작업을 뷰에서 주입시켜야한다. 그래서 뷰컨트롤러에서 newValue에 대해 처리하는 작업을 정의한다.

```swift
// 뷰컨트롤러에서 주입해주는 클로저로 뷰모델 내부 상태가 변했을 때 취해야 할 액션, "관찰"에 해당
// 이 예제에서는 DiffableDataSource의 SnapShot 변경
travelListViewModel?.didFetch = { [weak self] fetchedTravels in
    self?.applySnapShot(with: fetchedTravels)
}
```

매우 간단한 예제이지만 뷰와 데이터의 바인딩을 보기 위해 여러 파일을 넘나들어야하는 불편함이 있었다. 또한, 이런 데이터바인딩과 별개로 테이블뷰의 셀을 채우기 위해 UITableViewDataSource 프로토콜을 채택하여 여러 함수를 작성하기도 해야한다. RxSwift를 적용하면서 이런 불편함을 해소할 수 있었다고 생각한다.

## #UI컴포넌트 #RxCocoa

일반적으로 TextField, TableView 등 UI 컴포넌트와 데이터소스를 바인딩할 때 delegate 패턴을 사용한다. 예를 들어 테이블뷰의 셀의 내용을 데이터소스로 채울 때 UITableViewDataSource 프로토콜을 채택한다거나 SearchBar의 텍스트 변화를 감지하기 위해 UISearchBarDelegate 프로토콜을 채택하곤한다.

RxSwift를 도입하면 이런 과정 없이 매우 간결한 코드로 UI 요소와 데이터소스를 바인딩할 수 있다. 아래 예제를 통해 알아보자.

<img src = "https://user-images.githubusercontent.com/35067611/113367892-38ab6200-9398-11eb-96fb-ceaff0265d7a.gif" width="350px">

이 예제에서는 서칭 쿼리를 트래킹하기 위한 UISearchBarDelegate, 테이블뷰의 데이터를 채워넣기 위한 UITableViewDataSource 프로토콜을 전혀 채택하지 않고 Rx만을 이용해서 바인딩했다. 아래와 같이 입력/출력으로 스트림을 나누어 매우 직관적으로 바인딩할 수 있었다.

```swift
private let allCountries = ["Korea", "Japan", "China", "USA", "India"]
private lazy var showingCountries = BehaviorSubject<[String]>(value: allCountries)
private var searchText = BehaviorSubject<String>(value: "")

...

private func bindInput() {
    searchBar.rx.text.orEmpty
        .bind(to: searchText)
        .disposed(by: disposeBag)
}

private func bindOutput() {
    searchText
        .map { [weak self] query in
            guard let self = self else { return [] }
            return self.allCountries.filter({ $0.lowercased().hasPrefix(query.lowercased())})
        }
        .bind(to: showingCountries)
        .disposed(by: disposeBag)

    showingCountries
        .bind(to: tableView.rx.items(cellIdentifier: MyCell.identifier, cellType: MyCell.self)) { _, element, cell in
            cell.countryLabel.text = element
        }
        .disposed(by: disposeBag)
}
```

이처럼 UI 컴포넌트와의 바인딩 시 코드의 양을 줄이고 간결하게 표현할 수 있는 장점이 있다. RxSwift를 사용할만한 이유를 크게 네 가지 측면에서 살펴보았는데 모든 예시를 살펴보면 코드가 전부 선언적으로 작성되었음을 알 수 있다. RxSwift의 operator 사용이 고차함수 map, filter, reduce와 같이 선언적으로 되기 때문에 문맥을 파악하는 데에도 수월해진다는 느낌을 받았다.

이번 포스팅에서는 RxSwift를 사용하는 이유에 대해서 알아보았다. 예제로 작성한 코드가 많았는데 이 코드들 모두 RxSwift 1주일차가 작성한 코드이므로.. 좋은 코드는 아닐 수 있다.

## References

[ReactiveX 공식 홈페이지](http://reactivex.io)

[ReactiveX/RxSwift 공식 GitHub](https://github.com/ReactiveX/RxSwift)

[곰튀김님 - RxSwift 4시간만에 끝내기 시즌2 종합편](https://youtu.be/iHKBNYMWd5I)
