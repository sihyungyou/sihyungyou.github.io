---
layout: post
title: "Network : Intro 02"
tags: [한동대, 공부]
comments: true
---

> 1.3 Network Core  

### Network Core, Routers, and Switches
여러 edges(단말들)가 있다고 하자. 이들끼리 통신을 하려면 network core도 필요하지만 edge와 core를 이어주는 access망도 필요하다. 이 때 모든 users 끼리 1:1로 매칭하는 것은 어마어마한 연결을 필요로 한다. 이런 문제를 해결하기 위해서 연결이 필요할 때만 할 수 있도록 하는 router/switch의 개념이 등장한다. 그리고 access를 구현해주는 방식도 몇 가지 나온다. 가장 대표적이고 기본적인 두 approaches가 circuit switching, packet switching이다. 전자는 전화, 후자는 인터넷에서 각각 쓰이는 접근방식이다.  

### Circuit Switching (telecommunication networks)  
A와 B를 연결할 때 둘 사이의 최적경로를 찾는다. 그 최적경로에 놓인 선이 1M라고 할 때 그것을 64kbps로 나누어 A-B에 할당해준다. 이 때 이 경로는 온전히 A-B사이에게 소유되어 다른 사람이 동시에 사용할 수 없는데(resource reservation/dedicated resources) 쉽게 생각하면 혼자 사용할 수 있는 전용고속도로이다. Resource reservation은 각 call 마다 이루어진다. 이런 방식의 특징들은 다음과 같다.  
- transfer data to receiver at the guaranteed rate : packet 단위로 끊어서 store-and-forward 방식이 아니기 때문에 한번에 data를 전송한다.  
- no sharing : 이미 할당된 자원은 다른 user가 사용할 수 없다.  
- call setup : A-B사이의 최적경로를 찾고 resource reservation하는 데에 시간이 필요하다.  

