---
layout: post
title: "iOS) [번역] 코어데이터와 멀티스레딩"
tags: [iOS, Swift]
comments: true
---

> Teaching Core Data to dance with threads  

⚠ iOS알못의 글이므로 틀린 정보가 있을 수 있습니다.  

코어데이터에 데이터를 읽고 쓰는 것은 메모리에 동일한 작업을 하는 것보다 더 오래걸린다. 특히 데이터의 양이 많아지면 더욱 그렇다. 그래서 메인 스레드가 아닌 백그라운드 스레드에서 해당 작업을 진행하는 것이 좋다. 하지만 여기서 생각해야 할 점은 코어데이터는 thread-safe 하지 않다는 것이다. 이 문제에 대해 어떻게 해결하는지 여러 소스들을 찾아보다가 [좋은 글](https://duncsand.medium.com/threading-43a9081284e5)을 찾아서 번역(의역)을 해보기로했다!  

---  

코어데이터는 애플의 내장(inbuilt) 영속 데이터 API이다. 매우 좋은 API이지만 주의해서 사용해야 할 부분들이 있다.

일반적으로 코어데이터를 사용하는 iOS 앱은 멀티 스레드로 이루어졌을 것이다. 유저 인터페이스를 업데이트 하는 과정은 메인 스레드에서, 네트워크를 통해 얻은 데이터를 코어데이터에 저장하는 과정은 백그라운드에서 콜백을 통해 구현될 것이다.

하지만 코어데이터는 스레드 안정성을 보장하지 않는다. 그러므로 하나의 코어데이터 데이터베이스를 여러 스레드에서 접근하거나 코어데이터 객체(objects)를 여러 스레드간 공유하는 것은 안전하지 않다. 혹은 이런 스레딩 한계를 고려하지 않은 디자인 역시 안전하지 않다.

(대충 본인이 여러 방법을 간구하다가 찾은 솔루션을 소개하겠다는 내용, 중략)

### Background and Main Threads

위에서 언급했듯, 앱은 네트워크로부터 코어데이터 데이터베이스로 데이터를 로드하고, 유저 인터페이스에 데이터 정보를 보여주는데 이 과정은 멀티 스레딩으로 구현되었을 것이다. 올바른 스레딩 디자인이 아니라면 특정 시점에 앱은 크래시 될 것이다. 다행인 것은 코어데이터의 스레딩 전략은 이해하고 나면 매우 쉽다는 것이다.

### NSManagedObjectContext for each Thread

코어데이터 호출(calls)은 `NSManagedObjectContext`를 통해서 이뤄진다. 그리고 이 컨텍스트 객체는 스레드 간 공유가 불가능하다. 여기부터 시작해보자.

여러 구분된(separate) NSManagedObjectContext를 각각의 스레드에 생성함으로써 멀티 스레딩을 지원할 수 있다. 그리고 이 여러 컨텍스트들을 하나의 물리적인 데이터 저장공간을 관리하여 여러 스레드를 사용하더라도 하나의 통일된 데이터 저장공간과 통신할 수 있도록 해주는 `NSPersistentContainer`로 합치는 방법을 사용하는 것이다. 그렇다면 이것을 어떻게 할 수 있을까?

### Avoid Merge Conflicts with a single background NSManagedObjectContext

하나의 NSPersistentContainer에 여러 NSManagedObjectContext가 write을 수행한다면 병합 충돌(merge conflicts)가 일어날 가능성이 있다. 충돌이란 두 개의 다른 컨텍스트가 동일한 객체를 동시에 변경하는 것을 말한다. 이는 Merge Policy를 세팅함으로 해결 가능하다. (코어데이터가 이런 충돌이 일어났을 때 어떻게 해결해야 하는지에 대한 정책) 하지만 이는 매우 복잡하고 이런 정책이 항상 프로그래머의 생각대로 동작하지는 않는다.

따라서 이런 충돌을 가능하다면 피하는 방법이 더 좋다. 이것을 위한 간단한 전략이 있다.

모든 백그라운드 코어데이터 처리를 위한 하나의 컨텍스트만을 생성하는 것이다. 이로써 모든 코어데이터의 백그라운드 작업이 하나의 스레드(메인이 아닌)에서 수행될 수 있도록 하는 것이다. 이 방법은 모든 업데이트가 병렬이 아닌 한번에 하나씩(one-at-a-time) 처리되도록 보장하여 더욱 안전하다.

### Implementing a single background NSManagedObjectContext

애플공식문서에 따르면 코어데이터에서 백그라운드 작업을 수행하는 것은 아래 코드로 쉽게 구현이 가능하다.

```swift
let container = self.persistentContainer
container.performBackgroundTask() { (context) in
    // Do some core data processing here
    do {
        try context.save()
    } catch {
        fatalError("Failure to save context: \(error)")
    }
}
```

사실 여기에 쓰인 performBackgroundTask 메소드는 호출될 때마다 새로운 스레드를 생성(spawn)한다. 그래서 싱글 백그라운드 컨테스트를 생성하는 더 좋은 방법을 찾았다.

```swift
lazy var backgroundContext: NSManagedObjectContext = {
    let newbackgroundContext = persistentContainer.newBackgroundContext()        
    newbackgroundContext.automaticallyMergesChangesFromParent = true
    return newbackgroundContext
}()
```

이 코드는 코어데이터의 모든 백그라운드 업데이트가 하나의 스레드에서, 그리고 동일한 컨텍스트에서 수행됨을 보장한다. 모든 업데이트들은 순서대로 실행되므로 병합 충돌을 피할 수 있으며 그러므로 thread-safe 하다.

### Sharing data retrieved from Core Data across threads

때로는 코어데이터의 객체(object)들을 스레드끼리 넘겨주어야 할 때가 있다. 하지만 코어데이터 객체 또한 스레드 안정적이지 않기 때문에 이는 불가능하다.

하지만 더 간단한 방법이 있다. 모든 코어데이터 객체는 `objectID`를 갖고 있으며 이 정보는 thread-safe 하다. 그러므로 객체 자체 대신 이 id 정보를 스레드 간 서로 넘겨주는 것은 가능할 것이다.

id를 받은 스레드는 해당 정보를 사용해 컨텍스트로부터 객체를 읽으면(retrieve) 된다.

```swift
let object = try self.context.existingObject(with: objectID) as! Object
```

### Make sure your ObjectIDs are permanent

코어데이터 객체를 생성할 때 임시 id(temporary objectID)가 주어진다. 이것은 `isTemporaryID` 속성을 통해 테스트해볼 수 있다. true를 반환한다면 이 정보를 주고받아서는 안된다. 다른 스레드에서 임시 id 정보를 접근하려 한다면 대응되는 객체가 없을 수 있기 때문이다.

하지만 간단하게 이 문제도 해결할 수 있다. 이 objectID 정보를 활용해서 무언가를 하기 전에 항상 컨텍스트에서 `save` 메소드를 호출하는 것이다. 이는 객체에 영구적인 id(non-temporary objectID)를 부여하고, 스레드 간 이 정보를 전달해도 문제가 없다.

### Summary

1. 두 NSManagedObjectContext를 생성한다. 하나는 읽은 데이터를 보여주는 등의 UI 처리를 위한 메인 스레드, 다른 하나는 코어데이터 업데이트를 위한 백그라운드 스레드를 위한 것이다.
2. `self.backgroundContext.performAndWait { }` 메소드를 사용하여 모든 백그라운드 코어데이터 처리를 하나의 백그라운드 컨텍스트에서 처리되도록 한다.
3. 코어데이터 객체를 스레드끼리 전달하지 말고 objectID를 사용한다. 그리고 `context.existingObject(with: objectID)` 메소드를 통해 정보를 retrieve 해올 수 있다.
4. objectID를 넘겨주기 전에 반드시 컨텍스트 save를 한다.

## References

[Teaching Core Data to dance with threads](https://duncsand.medium.com/threading-43a9081284e5)
