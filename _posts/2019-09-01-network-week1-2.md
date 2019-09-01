---
layout: post
title: "Network : Intro 02"
tags: [한동대, 공부]
comments: true
---

> 1.2 Network edge  

### What is Network Edge?  
hosts : clients and servers (client는 service request를 하고 servers는 request에 response한다)  
P2P(peer to peer)도 있다. 이 경우, 두 peers가 대등하여 두 쪽 모두 client, server가 될 수 있다.  

### Access Networks  
How to connect end systems to edge router?  
residential access nets, institutional access networks, mobile access networks.. 하지만 문제는 결국 access망의 속도다. Core 망은 속도가 빠르다. 그러나 외부로 연결될 때는 이야기가 달라진다. 그래서 기지국에서 가정집으로 어떻게 선을 깔 것인지, 어떤 재료의 선을 사용할 것인지 등 여러 질문들이 나올 수 있다.  

옛날 가정집에서는 전화선과 인터넷선이 같았다. 이런 구조의 단점은 동시에 컴퓨터와 전화를 사용할 수 없다는 것이다. 그 이유는 음성정보와 data 정보의 주파수가 겹치기 때문이다. 둘 중 하나는 끊어져야 나머지 하나의 정보가 통신망을 통해 전달될 수 있었다. 그래서 splitter가 개발된다. splitter는 음성과 data정보를 다른 주파수로 transmit 하여 동시에 사용할 수 있게 해주었다.  

그리고 FTTH (Fiber To The Home) 라는 개념이 나온다. 이것은 집까지 광섬유(케이블)를 깔아주자는 것이다. 물론 가정마다 모두 광섬유 인터넷 선을 깔면 돈이 엄청나게 들기 때문에 여러 대안이 나오는데 여러 집들로부터 가까운 곳까지는 빠른 광섬유로 깔고 그 지점부터 각 가정까지는 좀 느린 선을 까는 AON(Actice Optical Network) 방식이 한 가지다. 이외에도 TDMA-PON(Passive), WDMA-PON(Wavelength Division Multiplexing) 등이 있다.  