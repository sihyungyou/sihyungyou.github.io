---
layout: post
title: "Network : Intro 01"
tags: [한동대, 공부]
comments: true
---

> 1.1 What is the Internet?, 1.2 Network edge 

### What are networks?  
the interconnection of a set of devices capable to carry information  

네트워크란 정보를 전달하기 위한 것들을 통칭하는 단어다. 그 종류는 여러가지가 있는데 전화 네트워크, 인터넷, 종합정보통신망 등 흔히 우리말로 "망"을 붙이곤 한다. 그렇다면 왜 이렇게 여러 종류의 네트워크가 있을까? 하나의 네트워크로 인터넷이든, 전화든 정보만 주고 받을 수 있으면 되는 것 아닌가?  

네트워크 종류가 여러개로 구분되는 이유는 전달하고자 하는 정보의 특성이 다르기 때문이다. 예를 들면 전화는 음성 정보를 전달한다. 아날로그 신호를 디지털 신호로 샘플링해서 보내는데 이런 특성을 가지고 잘 구현된 네트워크가 PSTN(Public Switched Telephone Network)인 것이다. 여기서 이루어지는 정보이동의 특징은 음성 정보를 일정하게 쭉 이어서 보낸다는 점이다. 반면, 인터넷은 데이터를 한번에 보내지 않는다. Packet이라는 단위로 묶고 끊어서 보낸다.  

위와 같이 네트워크의 종류를 구분하는 것은 특정 정보를 전달하는 데에 최적화할 수는 있겠으나 분명 힘든 작업이다. 이런 점을 보완하여 하나로 통합해 여러 서비스를 제공하는 목적으로 만들어진 것이 종합정보통신망(Integrated Services Digital Network)이다. 여기엔 N-ISDN(Narrowband)과 B-ISDN(Broadband)이 있는데 후자는 말 그대로 통합서비스를 광대역에 설치하는 이상적이고, 비싸고, 복잡한 꿈의 망이었다. 그래서 사라졌다.. 이로부터 우리는 system을 만들 때에 기억해야 할 교훈을 얻을 수 있는데 바로 KISS(Keep It Simple, Stupid)이다.  

### Terms  
`hosts`  
단말(end system)이라고도 한다. PC, server, laptop, smartphone 등 runninc network apps을 의미한다.  

`communication links`  
physical media used to connect two or more devices "directly"  
이 link가 쭉 연결된 것이 path이다. 이 때 연결은 말 그대로 물리적 연결을 의미하며 광선, 구리, 무선 등 여러 선택지가 있다. 이런 재료들 중 무얼 선택하느냐에 따라서 transmission rate (bandwidth) 즉, 정보전송속도의 차이가 있다.  

`routers (switches)`  
to forward packets  
packet을 전달한다. 스위치는 생소할 수 있는데, N명이 있고 그 N명끼리 모두 통신을 이으려면 nC2 = n(n-1)/2 개의 선이 필요하다. 즉 O(n^2)개인데 이는 너무 많으므로 이런 문제를 해소하기위해 나온 것이 스위치이다. 필요할 때만 통신될 수 있도록 돕는다.  

`Internetwork (internet)`  
network of networks : 독립적인 각각의 네트워크들을 연결시킨 것  
그러나 여러 네트워크들을 단순히 선으로 연결만 시킨다고 통신이 되는 것은 아니다. 각 네트워크들이 쓰는 방법들이 모두 다를 수 있기 때문이다. 이 때 필요한 것이 Router 혹은 Gateway 이다. 또한 인터넷에 대한 표준을 정의해야 한다. 그것을 Protocol이라고 하는데 수 많은 프로토콜이 있고 가장 대표적으로 TCP/IP가 있다. IP는 전쟁에서 길이 하나라도 있을 때 망끼리 통신할 수 있도록 하기 위한 배경에서 나온 임의의 네트워크를 연결하는 프로토콜이다.  

### The Internet Cannot Support Guaranted Service  
인터넷은 보장된 서비스를 지원하지 않는다. 보장된 서비스란 두 가지 측면에서 이해할 수 있는데 첫째 delay(time) 이다. packet을 보내면 언제 도착한다는 보장이 없다. 어떨 때는 delay가 있고, 어떨 때는 없다. 이 두 경우가 계속해서 interaction 한다는 것이 문제다! 두번째 측면은 bandwidth(throughput) 즉, 초당 data 전송량을 보장할 수 없다. 여기서 unreliable(data lost가 있을 수 있음), relialbe(connection이 끊어지지 않는 이상 safe. data lost의 경우 TCP 스스로 recover함)로 나뉘는데 TCP와 UDP를 예를들어 설명하도록 한다.  

IP외에 또 다른 유명한 프로토콜로는 TCP와 UDP가 있다. 둘의 차이점은 UDP의 경우 편지를 부친다고 생각하면 된다. 즉 상대방과의 연결을 요구하지 않고 (connectionless unreliable) 일방적으로 붙인다. 반면 TCP는 정보를 보내기 전에 사전초지가 필요하다. 즉, 상대방과 연결되어있음을 확인한 후에(connection-oriented reliable) 정보를 주고받는 전화통화 같은 과정이라고 이해하면 된다.  

### What is a protocol?  
A protocol is the set of rules and procedures to which the information exchange between two or more entities should adhere.  
말이 어렵지만 한 마디로 네트워크끼리 연결될 때 지켜져야 할 약속과 절차의 집합이라고 생각하면 되겠다. TCP에서 유명한 three way handshake라는 말이 있는데 처음에 connection을 만들 때 packet 세 개가 왔다갔다 해야 한다는 뜻이다. request, acknowledge, request back이 그 세 가지이다.  

### What is Network Edge?  
hosts : clients and servers (client는 service request를 하고 servers는 request에 response한다)  
P2P(peer to peer)도 있다. 이 경우, 두 peers가 대등하여 두 쪽 모두 client, server가 될 수 있다.  

### Access Networks  
How to connect end systems to edge router?  
residential access nets, institutional access networks, mobile access networks.. 하지만 문제는 결국 access망의 속도다. Core 망은 속도가 빠르다. 그러나 외부로 연결될 때는 이야기가 달라진다. 그래서 기지국에서 가정집으로 어떻게 선을 깔 것인지, 어떤 재료의 선을 사용할 것인지 등 여러 질문들이 나올 수 있다.  

옛날 가정집에서는 전화선과 인터넷선이 같았다. 이런 구조의 단점은 동시에 컴퓨터와 전화를 사용할 수 없다는 것이다. 그 이유는 음성정보와 data 정보의 주파수가 겹치기 때문이다. 둘 중 하나는 끊어져야 나머지 하나의 정보가 통신망을 통해 전달될 수 있었다. 그래서 splitter가 개발된다. splitter는 음성과 data정보를 다른 주파수로 transmit 하여 동시에 사용할 수 있게 해주었다.  

그리고 FTTH (Fiber To The Home) 라는 개념이 나온다. 이것은 집까지 광섬유(케이블)를 깔아주자는 것이다. 물론 가정마다 모두 광섬유 인터넷 선을 깔면 돈이 엄청나게 들기 때문에 여러 대안이 나오는데 여러 집들로부터 가까운 곳까지는 빠른 광섬유로 깔고 그 지점부터 각 가정까지는 좀 느린 선을 까는 AON(Actice Optical Network) 방식이 한 가지다. 이외에도 TDMA-PON(Passive), WDMA-PON(Wavelength Division Multiplexing) 등이 있다.  