---
layout: post
title: "Network : Ch2-2"
tags: [한동대, 공부, 네트워크]
comments: true
---

> 2.2 Web and HTTP  

### Web and HTTP  
웹페이지는 많은 objects로 이루어져있다. 이 때 object란 HTML file, JPEG image, Java applet, audio file 등 많은 것이 될 수 있다. 이 중 HTML file로 이루어진 웹페이지는 몇몇 referenced objects를 포함한다. 예를 들면 Hypertext/hpyermedia system (set of documents organized information)이 있으며 각 object는 URL에 의해 addressed된다.  

### HyperText Transfer Protocol Overview  
HTTP는 appliation layer의 protocol이다. 보통 application layer는 end user가 프로그램을 만들어서 서비스를 하는 것을 주로 생각하는데 사실 그것들은 "application layer에서 제공하는 service 위에서, 맞춰서" 작동하는 프로그램이다.  

아무튼! HTTP는 client/server architecture(model) 이다. request 하는 client는 웹 브라우저이고 Web server가 그 request들에 대해 어떻게 반응하는지를 규정하는 것이 HTTP 프로토콜이다.  

또한 HTTP는 stateless protocol이다. 지난 챕터에서 paket switching에서 잠시 소개되었던 개념인데 이미 지나간, 이전의 request에 대해서 정볼르 저장하고 있지 않다는 것을 의미한다. 기본적으로는 이전요청정보를 저장하지 않지만 요즘은 새로운 기술들이 추가로 들어간다. 예를 들어 특정 사용자가 어떤 웹 페이지에서 오래 머무르는지, 쇼핑을 하는지 등을 분석하여 사용자마다 페이지에 뜨는 사진, 정보가 달라진다.  

HTTP는 transport layer protocol로 TCP를 사용하며 well-known port number은 80번이다. TCP를 사용하니 reliable service를 제공할 것이다. HTTP-TCP도 non-persistent, persistent 두 가지 경우로 나뉠 수 있는데 이 둘의 차이점은 connection에 대해 자세히 보며 알아보자.  

### HTTP connections  
- Non-persistent HTTP  
간단히 줄이면 one connection, one object 라고 할 수 있다. 하나의 object는 "at most one object"를 의미한다. 하지만 한 번에 하나의 object를 보내는 connection이라도 여러개가 동시에 parallel하게 만들어지면 persistent connection 보다 response를 더 빨리 받을 수 있다. 단, 서버의 용량이 훨씬 많이 요구되고 무리가 간다.  

먼저 TCP 3-way handshaking 방식으로 세 개의 패킷이 왔다갔다 하면 connection이 만들어진다. 이는 두 방향으로 가는 독립적인 두 길이다. 그리고 하나의 object를 보낸 후에는 연결을 끊고, 다시 set up한다.  

- Persistent HTTP  
one connection, multiple objects 이다. 위의 non-persistent HTTP와는 다르게 한번 connection을 만들고 한 번 request를 하고 object를 보낼 때 연결을 끊고 다시 만들지 않는다. 물론 connection이 만들어지는 과정은 3-way handshaking 으로 같다.  

Persistent HTTP 방식에는 pipelining이라는 기술을 사용하기도 하는데, 이는 응답이 올 때 까지 기다리지 않고 request 전송이 끝났으면 바로 다음 request를 계속해서 보내는 것이다. 물론 효율이 더 좋아 HTTP/1.1 에서는 default이지만 거의 쓰지 않는다. 그 이유는 매우 복잡하고 request는 계속해서 연속적으로 보내지만 response를 보낼 때는 하나가 늦어지면 그 뒤의 정보가 준비되어있어도 늦어지는 문제가 있다. pipelining을 사용하지 않으면 RTT(round-trip time)당 한 object response를 받고 request를 한다.  


![Center example image](https://user-images.githubusercontent.com/35067611/65246627-edb95900-db29-11e9-8c6c-4e47760124c3.png "Center"){: .center-image}  
### HTTP Request Message  
HTTP Request Message는 ASCII로 human-readable format이다. 아래는 예시 사진이다.  
![Center example image](https://user-images.githubusercontent.com/35067611/65246627-edb95900-db29-11e9-8c6c-4e47760124c3.png "Center"){: .center-image}  
host 부분은 request일 때 반드시 필요한 부분이다.  
user-agent 부분은 browser dependency를 지원하며 user가 사용하는 브라우저 정보를 가지고 있다.  