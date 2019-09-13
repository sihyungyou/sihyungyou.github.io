---
layout: post
title: "Network : Intro 05"
tags: [한동대, 공부, 네트워크]
comments: true
---

> 1.5 Protocol layers, service models  

### Layered Systems  
네트워크는 매우 복잡하기 때문에 divide and conquer를 적용한 layering system으로 분할하여 설계한다. 여러 layer로 나누었을 때 갖는 중요한 특징은 각 layer가 독립적으로 디자인되고 구현될 수 있다는 점이다. 그러므로 각 계층의 detail(or technologyh)은 언제든지 독립적으로 수정되거나 고쳐질 수 있다. 물론 이런 계층화가 단점이 없지는 않다. 효율적인 측면에셔 보았을 때 네트워크를 통으로 만드는 것보다는 효율이 떨어진다.  

### Protocol Interfaces  
Protocol이란 같은 layer 안에 있는 것들 끼리 통신 하기 위한 것이다. 이 때 "같은 layer들 끼리"라는 말은 아래 그림과 같이 두 다른 host끼리 통신 시, 같은 레벨의 layer들 끼리의 통신을 의미한다. 한 호스트 내에서 다른 계층의 layer들 끼리 통신하는 interface를 service라고 부른다.  

![Center example image](https://user-images.githubusercontent.com/35067611/64670535-70924200-d4a0-11e9-859b-ccac2e29c5d3.png "Center"){: .center-image}  

### Protocol Interface and Services  
프로토콜이든 서비스든 정보가 왔다갔다 하는 "통신" interface이다. 즉 왔다갔다 하는 어떠한 정보의 약속된 단위(unit)이 각각 있을 것이다. 그것을 PDU, SDU라고 한다.  
PDU(Protocol Data Unit) : Header(PCI) + n-SDU
SDU(Service Data Unit) : (n-1)-SDU  

이렇게 정의만 써놓으면 무슨 말인지 알아듣기 쉽지 않으나 레이어끼리의, 혹은 호스트끼리의 통신이 이루어지는 것을 그림으로 보면 이해가 쉽다.  
![Center example image](https://user-images.githubusercontent.com/35067611/64670684-e696a900-d4a0-11e9-99a1-ec131b6a8bf0.png "Center"){: .center-image}  

위 그림에서 SAP란 Service Access Point로써 data를 교환하는 interface, 즉 service가 일어나는 포인트다.  
상위 layer에서 내려온 PDU는 하위 layer의 입장에서 SDU다. 하위 layer는 그 SDU정보를 해석하거나 열어보지 않고 거기에 자신이 해야할 일(예를 들면 transport)에 대한 정보만 덧붙이는데 이것이 header(PCI)이다. 그림에서 PDU가 내려와 SDU가 되고, 그 앞에 PCI가 붙어서 최종적으로 하위 layer입장에서 PDU가 만들어지는 과정이다.  

### Service Primitives  
Service Primitive Types는 서비스가 갖는 행위(?)의 타입들이다. 크게 네 가지로 구성되어있으며 protocol에 따라 네 가지 모두 필요한 것은 아니다.  
Request : an entity wants the service to do some work  
Indication : an entity is to be informed about an event  
Response : an entity wants to respond to an event  
Confirm : the response to an earlier request has come back  

위의 타입들을 실제로 사용할 때 PD-DATA.request, PD-DATA.confirm, .. 이런식으로 쓴다. 이 때 PD-DATA라는 service에 대해서 request, confirm 등을 하겠다는 의미이다.  

### OSI reference Model  
International Standards Organization(ISO)에서 만든 reference model로써 seven layer 구조이다.  
![Center example image](https://user-images.githubusercontent.com/35067611/64670975-ecd95500-d4a1-11e9-888c-0141383fbbab.png "Center"){: .center-image}  

1. Pysical layer : responsible for movements of individual bits from one hop(node) to the next  
2. Data link layer : reponsible for moving frames(link layer의 PDU) from one hop(node) to the next  
3. Network layer : responsible for the delivery of individual packets from the source host(컴퓨터) to the destination host  
4. Transport layer : responsible for the delivery of a message from one process(컴퓨터 안에 여러 프로세스) to another  
5. Session layer : responsible for dialog control(서로 언제 보내고, 받는지) and synchronization(file이 너무 크거나 길면 전송 중에 down되는 경우 있음. 이럴 때 중간부터 이어받는 mark point를 두고 일종의 동기화를 하는 것)  
6. Presentation layer : responsible for translation, compression, and encryption  
7. Application layer : responsible for providing services to the user  