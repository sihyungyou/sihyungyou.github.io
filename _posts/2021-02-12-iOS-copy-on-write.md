---
layout: post
title: "iOS) [번역] Swift의 Copy-on-Write 메커니즘"
tags: [iOS, Swift]
comments: true
---

> Understanding Swift Copy-on-Write mechanisms  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

오늘은 Swift의 Copy-on-Write(CoW)에 대한 [좋은 글](https://medium.com/@lucianoalmeida1/understanding-swift-copy-on-write-mechanisms-52ac31d68f2f)을 발견해 번역을 해보겠다!

---

Swift에는 참조타입(클래스)와 값타입(구조체, 튜플, 열거형)이 있다. 이 중 값타입은 copy semantic을 갖고 있다. 이 뜻은 만약 값 타입을 변수에 대입(assign)하거나 합수의 매개변수로 넘겨준다면(inout 매개변수가 아니라는 가정하에) 해당 값의 데이터가 모두 "복사"된다는 뜻이다. 동일한 콘텐츠의 두 값을 서로 다른 메모리 주소에 갖고 있게 될 것이다.

### What is this Copy-on-Write?

Swift에서 큰 값 타입의 데이터를 변수에 대입하거나 매개변수로 넘기게 되면 매우 비싼 복사 연산을 하게된다. 이 이슈를 최소화하기 위해서 Swift 표준 라이브러리는 배열과 같은 몇몇 값 타입에 대해 (두 개 이상의 참조가 있고 변형에 의해서만 복사가 일어나는) 하나의 참조만 있다면(uniquely referenced) 복사가 아니라 해당 참조 내에서 값 변경이 일어나는 메커니즘을 설계했다. 그래서 배열을 변수에 대입하거나 함수의 매개변수로 넘겨주는 것은 반드시 배열의 모든 데이터를 복사한다는 것을 의미하지 않는다.

한마디로 Copy-on-Write은 데이터 복사 시 실제로 값을 복사하지 않고, 동일한 값을 참조하다가 데이터 변경이 발생될 시에 복사해 값을 변경하는 기법이다.  

아래 예제를 보고 더 생각해보자.

```swift
import Foundation

func print(address o: UnsafeRawPointer ) {
    print(String(format: "%p", Int(bitPattern: o)))
}

var array1: [Int] = [0, 1, 2, 3]
var array2 = array1

//Print with just assign
print(address: array1) //0x600000078de0
print(address: array2) //0x600000078de0

//Let's mutate array2 to see what's
array2.append(4)

print(address: array2) //0x6000000aa100

//Output
//0x600000078de0 array1 address
//0x600000078de0 array2 address before mutation
//0x6000000aa100 array2 address after mutation
```

위 예제는 Copy-on-Write이 어떻게 작동하는지 보여주는 간단한 예이다. array1을 생성하고 array2에 대입했다. 이 시점에 Copy-on-Write 때문에 값의 복사가 아니라 array1과 array2는 동일한 메모리 공간을 가리키고(point) 있게 된다. 그리고 값의 변경(mutation)이 일어날 때에만(15-17 라인) 값이 복사된다.

### Implementing Copy-on-Write behavior for your custom value types

Copy-on-Write behavior를 직접 구현할 수도 있다. (프로그래머가 직접 구현하는 구조체 같은 값 타입의 경우 Copy-on-Write이 반영되어있지 않다)

```swift
final class Ref<T> {
    var val : T
    init(_ v : T) {val = v}
}

struct Box<T> {
    var ref : Ref<T>
    init(_ x : T) { ref = Ref(x) }

    var value: T {
        get { return ref.val }
        set {
          if (!isUniquelyReferencedNonObjC(&ref)) {
            ref = Ref(newValue)
            return
          }
          ref.val = newValue
        }
    }
}
// This code was an example taken from the swift repo doc file OptimizationTips
// Link: https://github.com/apple/swift/blob/master/docs/OptimizationTips.rst#advice-use-copy-on-write-semantics-for-large-values
```

이 샘플코드는 generic 값 타입 T에 Copy-on-Write을 어떻게 참조 타입을 사용하여 구현하는지 보여준다. 기본적으로 참조타입을 관리하고 하나의 참조만 있는 경우(uniquely referenced)가 아니라면 새로운 인스턴스를 반환하는 wrapper이다. 하나의 참조만 있다면 새로운 인스턴스를 생성하지 않고 참조타입의 값을 변형(mutate)시킬 뿐이다.

### Conclusion

Copy-on-Write은 Swift에서 매우 무거운(heavy) 연산인 값 타입의 복사를 최적화하는 매우 좋은 방법이다. 표준 라이브러리에 구현되어 있기 때문에 우리가 명시적으로 경험하는 일은 적다. 하지만 분명 이에 대해 이해하고 코딩을 하는 것은 매우 중요할 것이다.

## References

[Understanding Swift Copy-on-Write mechanisms](https://medium.com/@lucianoalmeida1/understanding-swift-copy-on-write-mechanisms-52ac31d68f2f)
