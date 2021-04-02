---
layout: post
title: "[번역] What is Reactive Programming?"
tags: [CS]
comments: true
---

> 반응형 프로그래밍이란 무엇인가  

Reactive Programming이 무엇인지 잘 설명해놓은 [글](https://medium.com/@kevalpatel2106/what-is-reactive-programming-da37c1611382)이 있어 번역해보았다!

---

요즘은 모두가 반응형 프로그래밍(Reactive programming)에 대해 이야기한다. 하지만 이 개념에 대해 여러번 들었으나 여전히 헷갈려서 좀 더 명확한 설명을 원하는 사람도 많다.

이 글에서는 반응형 프로그래밍의 기본 컨셉을 다룰 것이다. 그리고 다음 글부터 안드로이드 애플리케이션 개발에서 RxJava가 어떻게 사용되는지 실제로 프로그래밍 해 볼 것이다.

그렇다면 먼저 우리가 직면한 문제에 대해서 이해해보자. 우리는 왜 반응형 프로그래밍을 요구하는가? 아무 문제도 없다면 우리는 해결책도 필요없지 않은가?

## Why do we need Asynchronous work?

가장 간단한 답은 사용자 경험(user experience)를 개선하기 위해서이다. 우리는 우리의 애플리케이션이 더 반응적(responsive)이길 원한다. 메인 스레드가 freeze 되거나 속도가 느려지는 등의 성능이 아닌 부드러운 사용자 경험을 제공하기 원한다.

메인 스레드를 free 하게 유지하기 위해서 시간이 많이 소요되는 무거운 작업들을 background에서 해야할 필요가 있다. 또한 모바일 디바이스의 성능으로는 버거운 무겁고 복잡한 계산을 서버에서 하길 원한다. 그래서 네트워크 연산은 비동기가 필요한 것이다.

![1](https://user-images.githubusercontent.com/35067611/113369141-89708a00-939b-11eb-940d-27b8912eb6df.png)

모든 비동기 작업을 관리하는 라이브러리에서 필요한 것들에 대해 생각해보자. 우리는 비동기 라이브러리를 평가할 때 네가지 포인트를 생각할 수 있다. (사진에서 볼 수 있듯 evaluation matrix라고 한다)

- Explicit execution : 만약 새로운 스레드에서 많은 작업을 시작했다면 그것을 컨트롤할 수 있어야한다. 당신이 어떤 백그라운드 작업을 수행할 예정이라면 여러 정보를 모으고 그것을 준비한다. 그리고 준비가 되는대로 백그라운드 작업을 시작한다.
- Easy thread management : 비동기 작업에서는 스레드 관리가 핵심이다. 백그라운드 작업의 중간이나 끝에 메인 스레드에서 UI 업데이트를 해야하는 경우가 잦다. 그러기 위해서 백그라운드 스레드에서 메인 스레드로 작업을 넘겨야(pass) 할 때가 있다. 그러므로 스레드 간의 스위칭을 쉽게 할 수 있어야하고 필요하다면 작업을 다른 스레드로 넘겨줄 수 있어야한다.
- Easily composable : 이상적으로 백그라운드 스레드를 작동시키기 시작하면서 비동기 작업을 생성하는 것이 좋다. 다른 스레드에 의존하지 않고 해당 작업을 진행할 수 있으며 (특히 UI 스레드) 작업이 끝날 때까지 다른 스레드로부터 독립적이기 때문이다. 하지만 현실에서는 UI를 업데이트하고, 데이터베이스를 변경하는 등의 많은 스레드 의존적인 작업들을 해야한다.그러므로 비동기 라이브러리는 구성하기 쉬워야하고(easily composable) 에러를 덜 만들어야 할 것이다.
- Minimum the side effects : 멀티 스레드 환경에서 작업을 할 때 스레드 간 사이드 이펙트를 최소화하는 것이 중요하다. 이는 코드의 가독성을 높이고 훨씬 이해하기 쉽게 만들어 에러를 쉽게 추적할 수 있도록 도와준다.

## What is Reactive Programming?

위키피디아에 따르면 :

Reactive programming is a programming paradigm oriented around **data flows** and the **propagation of change**. This means that it should be possible to express static or dynamic data flows with ease in the programming languages used, and that the underlying execution model will **automatically** propagate changes through the data flow.

간단히 말해서, Reactive Programming 데이터 흐름은 한 구성 요소에 의해 방출되고 Rx 라이브러리에 의해 제공되는 기본 구조는 이러한 변경 사항을 등록된 다른 구성 요소에 전파할 것이다. 즉, Rx는 세가지 키 포인트로 이루어져있다.

Rx(Reactive Extensions) = Observable + Observer + Schedulers

이것들을 하나하나 더 자세히 살펴보자

- Observable : `Observable`은 데이터 스트림 그 이상 그 이하도 아니다. Observable은 여러 스레드 간 전달될 수 있는(can be passed around) 데이터를 감싼다(packs). 이 데이터 스트림은 그들의 라이프 사이클에서 주기적으로, 혹은 한번만 데이터를 방출한다(emit). 특정 이벤트에 기반하여 데이터를 방출할 수 있도록 도와주는 연산자(operators) 개념도 있는데 이는 아래에서 살펴보도록 하자. 지금은 observables를 suppliers라고 생각하면 된다. 이들은 데이터를 처리하고 다른 컴포넌트들로 제공한다.
- Observers : Observable이 데이터 스트림이었다면 `Observer`는 방출된 데이터 스트림을 소비한다(consume) Observers는 observable을 `subscribeOn()` 메소드를 사용하여 구독하고, 해당 observable로부터 방출되는 데이터를 받는다. observable이 데이터를 방출할 때마다 모든 등록된 observer들이 `onNext()` 콜백에서 데이터를 받는다. 여기서 JSON 파싱, UI 업데이트와 같은 여러 연산(operations)을 수행할 수 있다. 만약 observable에서 에러가 발생했다면 observer는 `onError()`에서 에러 정보를 받게 된다.
- Schedulers : Rx는 비동기 프로그래밍을 위한 것이며 스레드 관리가 필요하다는 것을 기억하자. 이 관점에서 scheduler가 등장한다. Scheduler는 Rx에서 observable과 observer에게 어떤 스레드에서 실행되어야 하는지 말해주는 컴포넌트이다. `observeOn()` 메소드를 통해 observer에게 어떤 스레드에서 실행되어야 할 지 말해줄 수 있다. 또한 `scheduleOn()` 메소드를 통해 observable에게 동일한 정보를 말해준다.

## 3 simple steps to use Rx in your application

![2](https://user-images.githubusercontent.com/35067611/113369148-8aa1b700-939b-11eb-88d6-54e5951a79b0.png)

간단한 예시를 살펴보자. 이 예시는 반응형 프로그래밍을 위한 세 가지 간단한 스텝을 소개한다. 언어는 RxJava이다.

```java
Observable<String> database = Observable      //Observable. This will emit the data
                .just(new String[]{"1", "2", "3", "4"});    //Operator

 Observer<String> observer = new Observer<String>() {
     @Override
     public void onCompleted() {
         //...
     }

     @Override
     public void onError(Throwable e) {
         //...
     }

     @Override
     public void onNext(String s) {
         //...
     }
 };

database.subscribeOn(Schedulers.newThread())          //Observable runs on new background thread.
        .observeOn(AndroidSchedulers.mainThread())    //Observer will run on main UI thread.
        .subscribe(observer);                         //Subscribe the observer
```

### Step-1 Create observable that emits the data:

위 예제에서 데이터베이스는 데이터를 방출하는 observable이다. 위 코드의 경우, 문자열 데이터들을 방출한다. `just()`는 매개변수에 제공된 데이터를 하나씩(one by one) 방출하는 operator이다. (just 외에도 여러 operator가 있으며 이에 대한 자세한 내용은 다음 글에서 볼 것이다)

### **Step-2 Create observer that consumes data:**

위 코드에서 observer 변수는 database observable이 방출하는 데이터를 소비하는 observer이다. observer는 받은 데이터를 처리하고 내부에서 에러를 관리하기도 한다.

### Step-3 Manage concurrency:

마지막 단계로, 병렬성을 관리하기 위한 scheduler를 정의한다. `subscribeOn(Schedulers.newThread())`는 observable에게 백그라운드 스레드에서 실행되도록 말하는 것이다. `observeOn(AndroidSchedulers.mainThread())`는 observer에게 메인 스레드에서 실행되도록 말한다. 이것이 반응형 프로그래밍의 기본이다.

## References

[What is Reactive Programming?](https://medium.com/@kevalpatel2106/what-is-reactive-programming-da37c1611382)
