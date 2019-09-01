---
layout: post
title: "Network : Intro"
tags: [한동대, 공부]
comments: true
---

> 1.1 What is the Internet?  

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
그러나 여러 네트워크들을 단순히 선으로 연결만 시킨다고 통신이 되는 것은 아니다. 각 네트워크들이 쓰는 방법들이 모두 다를 수 있기 때문이다. 이 때 필요한 것이 Router 혹은 Gateway 이다. 또한 인터넷에 대한 표준을 정의해야 한다. 그것을 Protocol이라고 하는데 수 많은 프로토콜이 있고 가장 대표적으로 TCP/IP가 있다. 