---
layout: post
title: "[번역] 다중상속 (1) - Diamond Problem"
tags: [iOS, Swift]
comments: true
---

> 다중상속의 문제점과 해결방법  

면접에서 다중상속에 대해서 질문을 받았는데 제대로 이해하고 있지 않아 대답을 하지 못했다. 그래서 복기를 하던 중 [좋은 글](https://medium.com/free-code-camp/multiple-inheritance-in-c-and-the-diamond-problem-7c12a9ddbbec)을 발견하여 번역을 해보려한다! 이 글은 다중상속의 문제점과 이에 대한 해결책으로 가상상속에 대해 간단히 언급하고 있다. 하지만 가상상속과 이런 점들을 Swift 언어 관점에 대해서는 서술하고 있지 않아 번역을 마치고 추가적으로 고민해본 점들을 정리할 예정이다.

---

다른 여러 `OOP` 언어들과 달리 C++는 다중상속을 지원한다. 다중상속은 자식 클래스가 하나 이상의 부모 클래스를 상속하도록 허용하는 것을 의미한다. 이는 매우 유용한 기능처럼 보이지만 프로그래밍 할 때 주의해야할 부분들이 있다. 아래의 예제를 보자.

```cpp
#include <iostream>

class LivingThing {
protected:
    void breathe() {
        std::cout << "I'm breathing as a living thing." << std::endl;
    }
};

class Animal : protected LivingThing {
protected:
    void breathe() {
        std::cout << "I'm breathing as an animal." << std::endl;
    }
};

class Reptile : protected LivingThing {
protected:
    void crawl() {
        std::cout << "I'm crawling as a reptile." << std::endl;
    }
};

class Snake : protected Animal, protected Reptile {
public:
    void breathe() {
        std::cout << "I'm breathing as a snake." << std::endl;
    }

    void crawl() {
        std::cout << "I'm crawling as a snake." << std::endl;
    }
};

int main() {
    Snake snake;

    snake.breathe();
    snake.crawl();

    return 0;
}
```

위 코드의 결과는 다음과 같이 나온다.

```
I'm breathing as a snake.
I’m crawling as a snake.
```

이 예제에서 기본 클래스는 `LivingThing`이다. 그리고 `Animal`, `Reptile` 클래스가 각각 LivingThing 클래스를 상속받고있다. 이 두 클래스 중 Animal 클래스만 breathe 메소드를 재정의(override)한다. 그리고 Snake 클래스는 Animal, Reptile 두 클래스 모두 상속받고 그들의 메소드들을 재정의한다. 이런 경우에는 문제가 없다.

하지만 약간의 복잡함을 더해보자. 만약 Reptile 클래스도 Animal 클래스처럼 `breathe()` 메소드를 재정의한다면 어떻게 될까? Snake 클래스는 어떤 breathe() 메소드를 호출해야할 지 결정하지 못한다. 이런 상황을 "`Diamond Problem`"이라고 한다.

![1](https://user-images.githubusercontent.com/35067611/108662978-d34f9000-7512-11eb-999d-0398c796a1a1.png)

### Diamond Problem

아래의 코드를 보자. 위에서 소개한 예제에 Reptile 클래스가 breathe() 메소드를 재정의한 경우이다.

```cpp
#include <iostream>

class LivingThing {
protected:
    void breathe() {
        std::cout << "I'm breathing as a living thing." << std::endl;
    }
};

class Animal : protected LivingThing {
protected:
    void breathe() {
        std::cout << "I'm breathing as an animal." << std::endl;
    }
};

class Reptile : protected LivingThing {
public:
    void breathe() {
        std::cout << "I'm breathing as a reptile." << std::endl;
    }

    void crawl() {
        std::cout << "I'm crawling as a reptile." << std::endl;
    }
};

class Snake : public Animal, public Reptile {

};

int main() {
    Snake snake;

    snake.breathe();
    snake.crawl();

    return 0;
}
```

이 코드는 컴파일에러를 일으키며 아래와 같은 에러메세지를 출력한다.

```
member ‘breathe’ found in multiple base classes of different types
```

이 에러가 다중상속의 Diamond Problem 때문에 발생하는 것이다. Snake 클래스가 어떤 breathe() 메소드를 호출해야할지 모르는 것이다.

첫번째 예제에서는 Animal 클래스만 breathe() 메소드를 재정의했고, Reptile 클래스는 하지 않았다. 그러므로 Snake 클래스에게 어떤 breathe() 메소드를 호출할지 결정하는 것은 어려운 일이 아니었다. 그리고 실제로 Snake 클래스는 Animal 클래스의 breathe() 메소드를 호출하는 것으로 상황이 종료되었다.

두번째 예제에서는 Snake 클래스가 Animal, Reptile 클래스로부터 breathe() 메소드를 상속받고있다. 그리고 Snake 클래스가 이 메소드를 재정의하지 않았으므로 ambiguity가 있는 것이다.

C++는 다중상속처럼 여러 강력한 기능을 제공한다. 하지만 제공되는 모든 기능을 사용하는 것이 필수는 아닐 것이다. 필자는 다중상속보다 가상상속을 선호한다. 가상상속은 Diamond Problem을 해결할 수 있는 방법으로 자식 클래스로 하여금 하나의 공통(common) 기본 클래스 인스턴스만 갖도록 보장한다. 다른말로, Snake 클래스는 LivingThing 클래스의 하나의 인스턴스만 갖게된다는 것이다. Animal, Reptile 클래스들은 이 하나의 인스턴스를 공유한다.

이 방법은 컴파일 타임의 에러를 해결해준다. 추상 클래스로부터 파생된(derived) 클래스는 반드시 기본 클래스에 정의된 pure virtual function을 재정의해야한다.

---

## 가상상속

위 글에서는 주로 다중상속을 다루었고 다중상속의 문제점인 Diamond Problem과 이를 해결하는 가상상속에 대해서는 아주 간단히 언급되었다. 가상상속을 아래 예제를 통해 더 자세히 알아보자

```cpp
class BaseIO {
public:
    int mode;
};

class In : virtual public BaseIO {
public:
    int readPos;
};

class Out : virtual public BaseIO {
public:
    int writePos;
};

class InOut: public In, public Out {
public:
    bool safe;
};
```

`virtual` 키워드를 통해 상속을 받으면 파생 클래스의 객체가 생성될 때 기본 클래스의 멤버는 오직 한번만 생성한다. 즉, 기본 클래스의 멤버가 중복하여 생성되는 것을 방지한다. 위 예제에서는 mode가 중복 생성될 여지가 있는 멤버일 것이다.

<img width="753" alt="2" src="https://user-images.githubusercontent.com/35067611/108662983-d64a8080-7512-11eb-8e14-62c10f7c558b.png">

왼쪽은 virtual 키워드 없이 다중상속을 했을 때, 그리고 오른쪽 사진이 Diamond Problem을 해결하는 가상상속을 적용한 경우의 메모리구조이다.

## 왜 다중상속을 지원하지 않는가?

Swift 뿐만 아니라 다른 여러 언어에서도 다중상속을 지원하지 않는 경우가 많다. 이는 Diamond Problem 때문인데 (물론 위에서 설명된 C++의 가상상속 등을 통해 해결할 수는 있지만) 이러한 문제점을 컨트롤 하는 것을 개발자에게 일임하지 않으려는 경향이 있어서 다중상속 자체를 불가능하도록 막아두는 것 같다. 이 외에도 클래스와 인터페이스(혹은 프로토콜)의 객체지향적 성격을 나누기 위한 의도도 있는 것으로 생각된다.

하지만 Java나 Swift 같이 다중상속을 막아놓은 언어에서도 Interface나 Protocol을 통해 다중상속의 기능은 흉내내면서도 Diamond Problem 같은 문제를 일으키지 않도록 지원한다. [다음 포스팅](https://sihyungyou.github.io/iOS-multiple-inheritance-2/)에서는 Swift의 Protocol은 어떻게 다중상속의 기능을 흉내내고 있는지 알아보도록 하겠다.

## References

[Multiple Inheritance in C++ and the Diamond Problem](https://medium.com/free-code-camp/multiple-inheritance-in-c-and-the-diamond-problem-7c12a9ddbbec)

[보충 수업 13 : 상속 기본 개념 - 다중 상속의 문제점](https://www.youtube.com/watch?v=kiab9vHBS48)
