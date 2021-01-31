---
layout: post
title: "iOS) PieChart 그려보기"
tags: [iOS, Swift]
comments: true
---

> 파이차트에 애니메이션을 적용해 그려보자!  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

오늘은 파이차트에 대해서 알아보자. 부스트포켓에서도 아래와 같이 파이차트를 애니메이션으로 그렸는데 이 과정에 대해 자세히 정리할 시간이 없어 미루고 미루다가 드디어 오늘 간단하게 프로젝트를 하나 생성해서 처음부터 로직을 따라가며 복기해보았다. 먼저 아래는 부스트포켓의 파이차트 애니메이션 결과물.

![boostpocket](https://user-images.githubusercontent.com/35067611/105623092-58834e80-5e5a-11eb-945a-cb55967c2fd8.gif)

## 기본적인 개념

위의 결과물을 보면 생각해볼만한 것이 두 가지 있다. 일단 원을 그리는 것과 각각의 슬라이스에 `UILabel`을 올리는 것이다. 이를 위해서 먼저 아래 두 그림에 대해 인지하고 코드를 봐야 이해가 보다 수월하다.

![1](https://user-images.githubusercontent.com/35067611/105623093-5ae5a880-5e5a-11eb-8c5c-04cf6c7d0114.png)

![2](https://user-images.githubusercontent.com/35067611/105623094-5b7e3f00-5e5a-11eb-9131-3ff814e1f4bd.png)

먼저 첫번째 사진과 아래 코드의 주석을 같이보자. `getDuration()` 함수는 전체 애니메이션 시간 중 각 slice의 비율만큼 할당하도록 계산하는 함수, `percentToRadian()` 함수는 각 slice의 퍼센트를 각도로 변환하는 함수일 뿐이니 지금은 신경쓰지 않아도 된다.

```swift
func addSlice(_ slice: Slice) {
    // CABasicAnimation을 선언하여 from, to, duration 등을 설정한다
    let animation = CABasicAnimation(keyPath: "strokeEnd")
    animation.fromValue = 0
    animation.toValue = 1
    animation.duration = getDuration(slice)
    animation.timingFunction = CAMediaTimingFunction(name: CAMediaTimingFunctionName.linear)
    animation.delegate = self

    // 실제로 stroke를 그릴 path를 UIBezierPath으로 선언한다
    let canvasWidth = self.frame.width * 0.8
    let path = UIBezierPath(arcCenter: self.center,
                            radius: canvasWidth * 3 / 8,
                            startAngle: percentToRadian(currentPercent),
                            endAngle: percentToRadian(currentPercent + slice.percent),
                            clockwise: true)

    // CAShapeLayer에 위에서 정의한 BezierPath와 animation을 넘겨준다
    let sliceLayer = CAShapeLayer()
    sliceLayer.path = path.cgPath
    sliceLayer.fillColor = nil
    sliceLayer.strokeColor = slice.color.cgColor
    sliceLayer.lineWidth = canvasWidth * 2 / 8
    sliceLayer.strokeEnd = 1
    sliceLayer.add(animation, forKey: animation.keyPath)

    self.layer.addSublayer(sliceLayer)
}
```

주목해서 볼 것은 그림에서 설명되었듯, `stroke path`와 그 선의 두께인 `lineWidth`이다. path를 BezierPath 객체로 선언해 정의하고 CAShapeLayer의 속성인 path에 넘겨주고 있다. 그리고 동일하게 CAShapeLayer에서 lineWidth를 정의해주는데 이 때 path의 반지름보다 조금 작게하여 가운데에 흰 원이 보이는 효과를 줄 수 있다.

두번째 사진과 관련한 `addLabel` 메소드 코드를 보자.

```swift
private func addLabel(_ slice: Slice) {
    let center = self.center
    // 현재 퍼센트와 그릴만큼의 퍼센트의 중간 위치를 찾고 거기에 레이블을 추가한다
    let labelCenter = getLabelCenter(currentPercent, currentPercent + slice.percent)

    let label = UILabel()
    label.textColor = .black
    label.numberOfLines = 0
    label.textAlignment = .center
    addSubview(label)

    let roundedPercentage = round(slice.percent * 1000) / 10
    label.text = "\(roundedPercentage)%"

    label.translatesAutoresizingMaskIntoConstraints = false
    NSLayoutConstraint.activate([label.centerXAnchor.constraint(equalTo: self.centerXAnchor, constant: labelCenter.x - center.x),
                                 label.centerYAnchor.constraint(equalTo: self.centerYAnchor, constant: labelCenter.y - center.y)])

    self.layoutIfNeeded()
}

private func getLabelCenter(_ fromPercent: CGFloat, _ toPercent: CGFloat) -> CGPoint {
    let canvasWidth = self.frame.width * 0.8
    let radius = canvasWidth * 3 / 8
    // 레이블이 위치해야할 곳을 계산하여 BezierPath로 나타낸다
    let labelAngle = percentToRadian((toPercent - fromPercent) / 2 + fromPercent)
    let path = UIBezierPath(arcCenter: self.center,
                            radius: radius,
                            startAngle: labelAngle,
                            endAngle: labelAngle,
                            clockwise: true)
    path.close()

    // BezierPath의 point를 반환
    return path.currentPoint
}
```

이렇게 레이블의 위치를 from과 to의 위치를 통해 계산하여 반환하고, 해당 위치에 UILabel을 생성해 addSubview 해주면 된다. 자세한 내용은 주석에 있으니 천천히 코드를 읽다보면 이해가 될 것이다.

## 순차적으로 그리기

위에서 코드로 본 addSlice, addLabel 메소드가 어떤 흐름으로 레이어를 그려내고 뷰를 추가하는지 이해했다면 이제 이 과정을 각 슬라이스마다 순차적으로 수행해주기만 하면 된다. 아래 코드를 따라가보면 어렵지 않게 반복되는 흐름을 이해할 수 있을 것이다.

```swift
// PieChartView 내 정의된 함수로, 처음 파이차트를 그리기 시작하는 순간에만 불린다.
func animateChart() {
    // 첫 슬라이스로 index를 설정하고, 현재까지 그린 percent도 0으로 설정한다
    sliceIndex = 0
    currentPercent = 0.0
    // 새로 animateChart를 할 때는 뷰컨에 이미 그려져있는 기존의 layer들과 label들을 모두 지워야한다
    self.layer.sublayers = nil
    removeAllLabels()

    if slices != nil && slices!.count > 0 {
        let firstSlice = slices![0]
        addSlice(firstSlice)
        addLabel(firstSlice)
    }
}

extension PieChartView: CAAnimationDelegate {
    // addSlice 함수가 끝나면 (그리는 작업이 끝나면) 불리는 함수로 다음 슬라이스가 있는지 확인한다
    // 있다면 다음 슬라이스에 대해 그리는 과정을 반복하는 함수
    func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
        if flag {
            // currentPercent는 현재까지 파이그래프를 그린 정도를 나타낸다
            // 여기에 현재 index번째의 slice percent만큼 그렸으므로 그 값만큼 더한다
            // "그렸으므로"인 이유는 현재 메소드가 불리는 시점이 index번째 슬라이스를 그리는 애니메이션이 didStop 된 이후이기 때문이다
            currentPercent += slices![sliceIndex].percent
            // 다음 슬라이스를 그려야하므로 sliceIndex를 한칸 옮겨준다
            sliceIndex += 1
            // 만약 다음 슬라이스가 남아있다면 addSlice를 반복한다
            if sliceIndex < slices!.count {
                let nextSlice = slices![sliceIndex]
                addSlice(nextSlice)
                addLabel(nextSlice)
            }
        }
    }
}
```

## 결과

![new](https://user-images.githubusercontent.com/35067611/105623090-55885e00-5e5a-11eb-912d-78d535c96a65.gif)

[전체 소스코드](https://gist.github.com/sihyungyou/9950aae64ff943cfaef7c00aa3c43ecf)

## References

[제드님 블로그 - UIBezierPath (5) - CAShapeLayer](https://zeddios.tistory.com/824)

[Animating Pie Chart on iOS in Swift](https://www.tnoda.com/blog/2019-06-18/)
