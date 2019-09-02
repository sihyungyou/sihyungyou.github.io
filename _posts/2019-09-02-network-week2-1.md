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
- link bandwidth, switch capacity :  

Circuit Switching은 다음의 세 가지 phase를 가지고 있다.  
- Circuit establishment (call setup) : end-to-end circuit must be established. resources on a path between end devices must be reserved.  
- Data transfer  
- Circuit release (disconnect) : after the data transfer, connection must be terminated so that the resources can be released.  

Circuit Switching의 장점은 delay가 거의 없다는 점이다. 물론 아무리 전기신호가 빠르다고 하지만 물리적 거리를 무시할 수 없기 때문에 propagation delay는 발생하지만 미미하다.  

### Multiplexing in Circuit-Switchied Networks  
위에서 1M의 link를 64kbps로 나누어 여러 end-to-end connection에 할당한다고 했다. (each link can be shared amont 'N' circuits) 할당하는 것에도 두 가지 방식이 있는데 FDM과 TDM이다. 기본적으로 single data link로 들어오는 multiple signals를 처리하기 위해서 multiplexing을 한다.  

- FDM (frequency division multiplexing)  
![Center example image](https://user-images.githubusercontent.com/35067611/64088073-b4879780-cd7a-11e9-959a-f0fc446bbd67.png "Center"){: .center-image}  

각 channel에 non-overlapped frequency bandwidth를 assign한다. 즉, 주파수가 서로 겹치지 않도록 modulation 하는 것이다. 하지만 전화의 경우 사람의 목소리는 대부분 동일한 주파수 대이다. 이렇게 어쩔수 없이 겹치는 경우에 modulate을 하여 주파수를 각기 다르게 만든다. 그리고 일부 frequency를 뽑아내서 다시 demodulation을 하여 receiver 에게 전달한다.  

- TDM (time division multiplexing)  
![Center example image](https://user-images.githubusercontent.com/35067611/64088094-d08b3900-cd7a-11e9-9a05-4d70ce664617.png "Center"){: .center-image}  

TDM은 digital multiplexing technique으로써 천천히 오는 것들을 시간을 기준으로 나누어 빠르게 보낸다.  
![Center example image](https://user-images.githubusercontent.com/35067611/64088163-28c23b00-cd7b-11e9-8f0a-2ecdc2f4ecb2.png "Center"){: .center-image}  

위 그림과 같이 들어오는 신호들의 각각 일부를 읽어서 전달하고 후에는 demux 한다. 이렇게 전달해도 data lost는 발생하지 않는다.  

### Circuit Switching advantages and disadvantages  
장점  
- Guaranteed quality of service : data can be transmitted at "fixed rate", no delay (negligible)  

단점  
- inefficient use of resources : channel capacity is dedicated even if no data is being transferred. 즉, constant bit rate으로 계속 data를 전송하면 효율이 100%겠지만 bursty한 transmit이라면 비효율적일 것이다.  
- circuit establishment delay : call setup 시간이 필요하다.  
