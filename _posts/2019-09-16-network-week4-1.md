---
layout: post
title: "Network : Ch2-1"
tags: [한동대, 공부, 네트워크]
comments: true
---

> 2.1 Principles of network applications  

### Network Application  
Network application은 서로 다른 end systems 위에서 도는 프로그램이다. 이들은 네트워크를 통해서 소통한다. 주의해야 할 것은 network core 부분에서 도는 application은 없다는 점이다. Network core 에서는 level 3 까지만 해서 packet을 목적지로 전달하는 역할을 담당한다. 즉 network core devices는 application layer에서 기능하지 않는다. 이러한 디자인이 application development의 속도를 빠르게 도와준다.  

### Application Architectures  
Application 구조에는 크게 세 가지가 있다.  
1. Client-server  
2. Peer-to-peer (P2P)  
3. Hybrid of client-server and P2P (P2P에 가까우나 P2P에서 하지 못하는 부분을 client-server 구조로 보완함)  

### Client-server  
먼저 용어 정리를 하자면 client와 server는 각각 정보가 없어서 "요청"하는 쪽, 정보를 가져서 "제공"하는 쪽이다. 일반적으로 서버가 physically 큰 부분을 차지한다고 해서 물리적 크기가 크고 작음이 client, server를 구분하는 것은 아니다. 예를 들어 IoT의 경우를 생각해보면 집의 온도를 측정하는 sensor가 있고 그 데이터를 받는 monitor가 있을 것이다. 이 때 서버는 물리적으로 큰 monitor가 아니라 온도라는 데이터를 요청받아 제공하는 온도센서이다.  

- Server  
항상 켜져 있다 (always-on host)  
고정 IP를 가지고 있다 (permanent IP address) 여러 client가 접속해야 하는 서버가 유동적 IP를 가지고 있다면 곤란할 것이다.  
여러 서버가 host에서 돌도록 server farms가 있다 (읭?)  

- Client  
서버와 소통한다.  
유동적 IP를 가지고 있다 (고정이 아니어도 된다)  
client-client끼리 직접적 소통을 하지 않는다 (허용되지 않는다)  

이런 Client-Server 구조는 한계점을 가지고 있는데 확장성 문제이다. Client 수가 늘어날 수록 서버 수도 늘려야 하는 것이다. 이런 문제를 해결하고 보완하고자 고안된 것이 P2P 구조이다.  

### Pure P2P  
위 Client-Server 구조의 확장성 문제를 해결하기 위한 P2P는 `Self-scalability`를 가진다. Service, request 모두 하는 peer들을 분산시켜놓는 것이다. 직관적인 예로는 모든 데이터가 서버에 저장되어있는 것이 아니라 모든 유저가 upload와 download를 모두 함으로써 데이터의 교환이 이루어진다.  

P2P는 서버가 항상 켜져있는 것은 아니다. 그리고 client끼리 directly 소통한다 (각 peer들 모두 service & reuqest 모두 한다, 즉 모두가 server이가 client의 역할을 한다)  

이러한 장점이 P2P의 단점이 되기도 한다. Peer 들은 계속 연결되어있지도 않고, IP도 계속 변하기 때문에 관리가 매우 어렵다. 또 이런 complex management 문제를 해결하기 위해 고안된 구조가 세번째 구조인 Hybrid of Client-server and P2P 이다.  

### Hybrid of Client-server and P2P  
말 그대로 하이브리드, Client-server 방식과 P2P 방식을 합친 것이다.  

Centralized server : 서버에 접속해서 IP를 알려준다 (Directory service)  
Client-client : (server on 확인 후) 통신은 direct하게 P2P방식으로 한다.  

ex) Skype, instant messaging  

### Processes Communicating  
Process : program running within a host (돌고 있는 프로그램)  

같은 host에서 process communication 은 inter-process communication defined by OS를 사용한다.  
다른 host의 process끼리의 communication은 exchanging message를 통해 이루어진다.  

client process : process that initiates communication (요청)  
server process : process that waits to be contacted (기다림)  

### Sockets  
정형화, 규정화된 형태로 "꽂아 쓸 수 있는 것", "끼우는 것"을 socket이라고 한다.  

Socket interface : application layer와 TCP/UDP와 같은 transport layer (protocol stack) 사이에 위치하며 process들이 socket을 통해 message를 주고 받는다.  

![Center example image](https://user-images.githubusercontent.com/35067611/64934473-ff7ad200-d885-11e9-861a-ce2192ce9d41.png "Center"){: .center-image}  

### Addressing Processes  
하나의 host는 여러 process를 가질 수 있다. 그렇다면 host의 IP주소를 알아도 그 중 어떤 process에게 전달하고자 하는지 정보가 충분치 않은 것이다. 그래서 추가적으로 필요한 정보가 port number 이다.  

Well-known port number : 유명한 service를 제공하는 port는 컨벤션으로 정해져있다. 예를 들어 웹서버인 HTTP server는 80번, email 서버인 SMTP는 25번이 있다.  

![Center example image](https://user-images.githubusercontent.com/35067611/64934526-5aacc480-d886-11e9-8d16-992c7047f20a.png "Center"){: .center-image}  

하나의 socket에 하나의 port number가 assign되어있다. 
IP address의 header에 transport protocol이 기록되어있다. 그것을 가지고 TCP/UDP 등 protocol을 결정하여 정보를 보낸다. 만약 TCP를 선택했다고 하면 TCP를 쓰는 여러 process가 있을 것이다. 그 여러 process 중 port number로 하나를 결정하여 socket을 통해 데이터를 전달한다. 위와 같은 결정요인들을 demultiplexing key라고 한다 (port #, ip address, transport protocol ..) "분산"시킬 때 어디로 보낼지 결정하기에 demux key 이며 encapsulation 할 때 header에 추가된다.  

