---
layout: post
title: "파이썬 시작하기 - 자료형1"
tags: [파이썬, 언어, 공부]
comments: true
---

> reference : 점프 투 파이썬 (https://wikidocs.net/book/1)  

요즘 대세는 파이썬이라고들 한다. 배우기 쉽고, 적용하기 쉬운 언어이고 실제 기업에서도 많이 쓰이기 떄문이다. 나도 교양수업으로 정확히 기억은 안나지만 1, 2년 전쯤 수업을 들었다. (주입식 교육의 폐혜인지 1도 기억이 나질 않는 게 함정이다..) 최근 해커톤 참여도 그렇고 머신러닝에서 필수적으로 쓰이는 등 여기저기서 파이썬 파이썬 하는 것 같다. 방학도 했고 슬슬 취업준비를 위해 알고리즘 공부도 해야하기에 파이썬을 다시 복습하려 한다. 지인이 추천해준 점프 투 파이썬이라는 튜토리얼을 보며 하나하나 개념을 복습하고 어느정도 복기가 된 후에는 파이썬으로 백준 문제를 풀어볼 생각이다. 첫번째 포스팅의 주제는 data type 이다.  

### no data type?  
사실 '자료형이 없다' 라는 것은 말이 안된다. data를 다루어야 하는데 그것들의 type이 나뉘어있지 않다면 어떻게 프로그래밍을 할 수 있겠는가! 다만, 파이썬에서는 여러 다른 자료형을 변수에 store하거나 handle 할 때에 '자료형 구분 없이' 한다는 것이 강점이다. 파이썬에도 당연히 정수, 실수, 배열 등 여러 자료형이 있으며 각각을 manage하는 함수들도 무수히 많다. 그 중 자주 쓸만한 것들을 정리해보려 한다. 이 블로그는 친절히 파이썬을 하나하나 알려주는 것이 아니라 내가 몰랐던 것 위주로 정리하는 매우 불친절한 블로그이므로 내 마음대로 내용을 빼고 더할 예정이다.  

### Number  
정수, 실수, 8진수, 16진수  
`+`, `-`, `*`, `/`, `%` 외에 흥미로웠던 것 : `**` (제곱), `//` (몫)  

### String  
"this is string", 'this is also string', """another one!""", '''the last one'''  
위와 같이 여러 방법으로 문자열을 나타낼 수 있는데 굳이 이렇게 네 가지 방법이나 가능한 이유를 오늘에서야 알았다(?)  
먼저 C에서 반드시 \" 로 표기해야했다면 파이썬에서는 " " 내에서는 자유롭게 ' 문자를 쓸수 있고 반대의 경우도 가능하다. 따옴표를 세 번이나 쓰는 것은 여러줄인 문자열을 변수에 대입하는 용도이다.  

문자열 끼리 더하고(concatenation), 곱하는 것(multiplication)이 가능하다.  
더하여, 문자열 길이를 구하는 내장함수가 구현되어 있다.
~~~python
a = "guess my length"
len(a)
~~~

string indexing, slicing은 자주 써보면 익숙해 질 것 같다.
formatting이 흥미로웠다!  
숫자 바로 대입  
~~~python
"i am %d years old" % 26  
~~~
문자열 바로 대입  
~~~python
"my name is %s" % "Jason"  
~~~
숫자 변수 대입  
~~~python
year = 2020
"i am class of %d" % year
~~~
두개 이상의 값 넣기  
~~~python
name = "Jason"
age = 26
"my name is %s and %d years old" % (name, age)  
~~~
f 문자열 포매팅 (파이썬 3.6 이상)  
~~~python
name = "Jason"  
age = 26  
f"my name is {name} and {age} years old"  
~~~
이 외에도 여러 경우의 포맷팅이 있는데 모든걸 외우고 있기보다는 필요할 때 찾아 쓰면 편할 것 같다.

매우매우 유용할 것 같은 문자열 함수들
1. count  
~~~python
a = "hobby"  
a.count('b')  
결과 : 2  
~~~

2. find  
~~~python
a = "Python is the best choice"
a.find('b')
결과 : 14
~~~

3. index  
~~~python
a = "Life is too short"  
a.index('t')  
결과 : 8  
~~~

4. upper  
~~~python
a = "hi"  
a.upper()  
결과 : 'HI'  
~~~

5. lower  
~~~python
a = "HI"  
a.lower()  
결과 : 'hi'  
~~~

6. split  
~~~python
a = "Life is too short"  
a.split()  
결과 : ['Life', 'is', 'too', 'short']  
b = "a:b:c:d"  
b.split(':')  
결과  :['a', 'b', 'c', 'd']  
~~~

7. replace  
~~~python
a = "Life is too short"  
a.replace("Life", "Your leg")  
결과 : 'Your leg is too short'  
~~~

이제부터는 진짜 파이썬에서 쓰이는 리스트, 튜플 등의 자료형인데 한 포스팅에 정리하기에 너무 길어져 다음 포스팅으로 넘기려 한다..!