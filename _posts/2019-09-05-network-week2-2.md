---
layout: post
title: "Network : Intro 03"
tags: [한동대, 공부, 네트워크]
comments: true
---

> Datagram Networks & Virtual Circuit Networks  

### Datagram Networks  
Packet Switch 중 한 방식으로 connection setup을 요구하지 않는다. 편지를 보내는 것과 같은 방식으로 목적지(수신자)정보를 가지고 있다. 모든 패킷들은 complete destination address를 가지고 있으며 그 주소는 globally unique identifier이다. 모든 패킷이 unique해야 하며 그러기 위해서는 이것들을 관리하는 주체가 필요하다. 아이피 주소를 전세계가 다르게 할당받아 쓰는 것과 같다. 모든 패킷은 routing table에 기반하여 independently forward 되며 같은 목적지를 가진 다른 패킷들이라도 다른 route를 가질 수 있다. 또한 Stateless router로써 router가 다운됐다가 다시 복구되었을 때 끊어졌을 당시에는 data loss가 있지만 연결들(links)은 다시 살아난다.  

![Center example image](https://user-images.githubusercontent.com/35067611/64339921-f71dce00-d01f-11e9-8830-0fdf0f37f6dd.png "Center"){: .center-image}  

Datagram Networks의 구조. 도착하는 순서가 반드시 패킷의 출발순서와 같지는 않기 때문에 reordering이 필요하다.  


### Virtual Circuit Networks  
Packet Switching과 Circuit Switching의 특성을 섞어놓은 듯한 방식이다. Data transfer에 앞서 sender-destination 사이의 virtual connection을 만들고 전송을 시작한다. 여기서 만든 연결이 곧 virtual circuit인 것이다. 각 switch는 connection state를 저장하고 있다. 즉 위의 Datagram Networks와 다르게 stateful 방식으로, 연결이 끊어졌다가 복구되었다면 routing table을 다시 setup해야 한다. (그 전의 routing table 정보는 날아간 것)  

Connection setup에는 signaling protocol이 사용되는데 이것 없이 장기적(일정시간)으로 쓸 bandwidth를 만들어달라고 전화국(?)에 요청하면 manually 만들어주기도 한다. 이를 PVC(permanent VC)라고 한다.  

![Center example image](https://user-images.githubusercontent.com/35067611/64340709-c3dc3e80-d021-11e9-85a8-cab5cbb9f0e7.png "Center"){: .center-image}  

Virtual Circuit Networks의 구조. 각 data header에 있는 VCI는 가변적이다. 이렇게 변하는 이유는 small look-up table을 유지하기 위함이다. (fast packet switching) 물론 이런 노력이 하드웨어딴에서 searching 하는 방식으로 매우 빨라진 요즘은 무색해지고 있다.  

Virtual Circuit Networks의 장점은 possibility to support QoS이다. Resource가 call setup 동안 할당되기 때문이다. 또한 패킷의 경로가 모두 table에 의해 정해져있으므로 In-order packet delivery가 가능하다.  

단점으로는 당연히 call setup procedure가 필요하다는 것이다. 또한 State-info를 저장하고 있는 stateful 방식이라는 점이다.  