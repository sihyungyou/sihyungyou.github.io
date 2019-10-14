---
layout: post
title: "Klaytn 정복기 04 : 네트워크 구조 2"
tags: [해커톤, 블록체인, Klaytn]
comments: true
---

> Klaytn Network 2  

### 이더리움 네트워크  
이더리움은 단일 네트워크로 구성원간에 구분이 없다. 누구나 블록 생성이 가능하고 블록을 만들었을 때 내가 먼저 만들었다고 가장 빨리, 널리 알려야한다. 그리고 블록 추가에 성공하면 보상을 받는 구조다. 이게 바로 PoW(Proof of Work) 작업증명 방식이다.  

마이닝 노드는 블록을 채굴하고 네트워크에 전파한 노드를 뜻하는데 이 노드가 가 누가될지 모르기 때문에 최대한 많은 곳에 붙어야 한다. 즉, 블록을 쓰는 노드는 최신정보를 갖고있고 내가 빨리 이 정보를 받아야 하는데 이게 어디있는지 모른다는 것이다. 만약 항상 A블록이 쓴다면 그 옆에만 달라붙어있으면 되지만 그게 아니다. 그 역할을 담당하는 노드가 계속 바뀌어서 최대한 많은 노드에 붙어서 전파 받아야한다. 즉 최대한 많은 노드와 연결하고 블록전파에 힘을 써야한다.  

### 클레이튼 네트워크  
클레이튼은 두개의 레이어를 가진 네트워크다. 매 라운드마다 합의노드들 중 하나가 뽑혀서 블록을 채굴하므로 빨리 정보를 받으려면 그 옆에 붙어야 한다. 이 때 코어셀 안에 있는 합의노드들이 블록을 만든다는 사실을 EN들이 알고있으니까 옆에 바로 붙는다. 그래야 신뢰도 높은 정보를 불러오거나 쓸 수 있기 때문이다.  

![Center example image](https://user-images.githubusercontent.com/35067611/66629167-efaf9d00-ec3b-11e9-867e-f487917a8f15.png "Center"){: .center-image}  

서버가 바로 CN과 연결 안되니까 내 컴퓨터를 EN화 시켜서 CN에 연결한다. 단, EN을 운영하는게 쉽진 않다. 모든 블록을 동기화 시켜야하기 때문에 새 블록 추가될때마다 동기화를 해야한다. 이런 문제 때문에 신뢰가능한 외부노드에 연결할 수 있다. 예를 들어, 웹개발자가 클레이튼에 연결해서 data read 하고싶다면 개인적인 EN을 운영하는 것보다 신뢰가능한 외부 공용 노드에 연결해서 쓰는게 훨씬 편하고 시간 절약도 될 것이다. 클레이튼은 두개의 레이어가 서로 신뢰하는 네트워크이고 내부 블록체인에 접근할 때 EN이 CN에 연결해서 빠르게 data read/write 가능한 구조다.  

출처 : [인프런 클레이튼 강의](https://www.inflearn.com/course/%ED%81%B4%EB%A0%88%EC%9D%B4%ED%8A%BC#)  