---
layout: post
title: "파이썬 시작하기 - 자료형2"
tags: [파이썬, 언어, 공부]
comments: true
---

> reference : 점프 투 파이썬 (https://wikidocs.net/book/1)  

### 파이썬 자료형 정리 두번째 포스팅  

### List  
파이썬에서 리스트는 이미 익숙한 개념인 배열이다. 다만 훨씬 편하다. indexing, slicing 함수가 배열을 다루는 데에 큰 도움을 주고 특히 list 안에 list가 들어가도 무관하다.  
slicing 은 string에서와 같은 문법으로 작동한다.  
concatenation, multiplication, length 구하는 것 또한 string과 같은 방식이다.  

list element 삭제는 del 함수로 한다.  
~~~python
a = [1, 2, 3]  
del a[1]  

결과 : [1, 3]  
~~~  
유용한 함수들
a.append(x) : 리스트의 맨 마지막에 x를 추가  
a.sort() : 리스트의 요소를 순서대로 정렬  
a.reverse() : 리스트를 역순으로 뒤집음  
a.index(x) : 리스트에 x 값이 있으면 3의 위치 값을 반환  
a.insert(a,b) : 리스트의 a번째 위치에 b를 삽입  
a.remove(x) : 리스트에서 첫 번째로 나오는 x를 삭제  
a.pop() : 리스트의 맨 마지막 요소를 돌려주고 그 요소는 삭제  
a.count(x) : 리스트 안에 x가 몇 개 있는지 조사  
a.extend(x) : x에는 리스트만 올 수 있으며 원래의 a 리스트에 x 리스트를 더함  

### Tuple  
element의 삭제, 변경 등이 불가한 것을 제외하면 list와 동일함!  

### Dictionary  
주의사항 : key 는 고유한 값이므로 중복되는 key의 경우 나머지 value는 무시된다!  

유용한 함수들  
dict.keys() : key들을 list로 만듦
dict.values()  : value들을 list로 만듦  
dict.items() : key - value 쌍 return  
dict.clear() : 모두 지우기  
dict.get(key) == dict[key] : 해당 key의 value return  

### Set  
* 중복을 허용하지 않는다.  
* 순서가 없다.  

유용한 함수들  
s1 & s2 or s1.intersection(s2) : 교집합  
s1 | s2 or s1.union(s2) : 합집합  
s1 - s2 or s1.difference(s2) : 차집합  
s2 - s1 or s2.difference(s1) : 차집합  
s1.add(x) : 1개의 값 집합에 추가  
s1.update([1, 2, 3]) : 여러개의 값 집합에 추가  
s1.remove(x) : x element 제거  