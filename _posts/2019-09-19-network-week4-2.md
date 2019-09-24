---
layout: post
title: "Network : CH 2-2"
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

먼저 TCP 3-way handshaking 방식으로 세 개의 패킷이 왔다갔다 하면 connection이 만들어진다. 이는 두 방향으로 가는 독립적인 두 길이다 (request, response 길이 각각 따로있다?). 그리고 하나의 object를 보낸 후에는 연결을 끊고, 다시 set up한다.  

- Persistent HTTP  
one connection, multiple objects 이다. 위의 non-persistent HTTP와는 다르게 한번 connection을 만들고 한 번 request를 하고 object를 보낼 때 연결을 끊고 다시 만들지 않는다. 물론 connection이 만들어지는 과정은 3-way handshaking 으로 같다.  

Persistent HTTP 방식에는 pipelining이라는 기술을 사용하기도 하는데, 이는 응답이 올 때 까지 기다리지 않고 request 전송이 끝났으면 바로 다음 request를 계속해서 보내는 것이다. 물론 효율이 더 좋아 HTTP/1.1 에서는 default이지만 거의 쓰지 않는다. 그 이유는 매우 복잡하고 request는 계속해서 연속적으로 보내지만 response를 보낼 때는 하나가 늦어지면 그 뒤의 정보가 준비되어있어도 늦어지는 문제가 있다. pipelining을 사용하지 않으면 RTT(round-trip time)당 한 object response를 받고 request를 한다.  

![Center example image](https://user-images.githubusercontent.com/35067611/65246627-edb95900-db29-11e9-8c6c-4e47760124c3.png "Center"){: .center-image}  

### HTTP Request Message  
HTTP messages 에는 request, response 두 가지 타입이 있다. 여기서 말하는 message란 어떠한 데이터인데 HTTP가 속한 application layerd의 TDU가 되겠다. 이 데이터는 ASCII(human readable) format이다.  

![Center example image](https://user-images.githubusercontent.com/35067611/65488147-654d0680-dee3-11e9-89e6-1a397a6b4689.png "Center"){: .center-image}  
위 사진은 request message의 일반적인 형태를 설명하고 있다.  

Request line은 method(GET, POST, PUT, DELETE 등), url(주소), version(HTTP v)을 포함한다.  
Header line 에는 주소(url)를 담은 host, browser dependency를 지원하며 user가 사용하는 브라우저 정보를 가지고 있는 user-agent, connection이 non-persistent인지 등의 정보, 언어정보 등이 담겨있다.  

방금 언급한 methods를 자세히 들여다보면 Resful architecture 라는 걸 볼 수 있다. 즉, CREATE(POST), READ(GET(PUT), DELETE(DELETE)의 스타일을 갖는 구조로 웹서비스가 구현된다. 최근에는 이에 더하여 NOTIFICATION(request를 하지 않아도 알림을 주는) 기능도 서비스에 많이 추가가 된다.  

### HTTP Response Message  
![Center example image](https://user-images.githubusercontent.com/35067611/65530938-d6b4a580-df33-11e9-86cf-8c61e62411f2.png "Center"){: .center-image}  
Request message에 대한 응답인 Response message이다. status line, header lines, data를 돌려주고 있다. 여기서 응답의 상태를 알려주는 status code는 익숙한 것들도 많다. 이 코드는 세자리 정수로 2XX은 OK, 3XX은 redirection, 4XX은 client error (우리가 흔히 아는 404 Not found, 400 bad request 등), 5XX는 server error 등이 있다.  

### User-server state : Cookies  
HTTP는 stateless 라서 user preference를 알 리가 없다. 그래서 추가로 그러한 정보를 저장하고 이용하기 위해 cookies라는 것이 등장했다. 쿠키는 state를 저장한다.  
서버가 새로운 유저를 인식하면 reponse에 set-cookie header를 추가한다 (서버쪽에 그 유저에 대한 정보를 저장한다). 그리고 다음에 client가 같은 서버에 접속했을 때 이전에 저장했던 쿠키정보를 포함시켜서 서비스를 한다. 즉, server가 response로 set-cookie, client가 request로 cookie 헤더를 포함한다. 

쿠키는 작은 데이터로 실행가능한 파일은 아니기 때문에 컴퓨터에 해를 끼치는 바이러스와 같은 개념은 아니다. 단, 개인정보 문제가 있을 수 있다. 

### Web caches (Proxy server)  
캐시는 중간에 있는(intermediary entity) web server라고 생각하면 된다. 접속하려 하는 origin server를 대신해서 request에 응답해준다. 이런 구조의 장점은 빠른 응답이다. 또한 접속 traffic을 줄일 수 있다. 단점으로는 캐시되어있지 않은 경우에 한해서 서버를 한 번 더 거치는 속도 저하가 있다.  

또한, 캐시서버에 있는 정보는 origin server에서 이루어지는 업데이트 사항을 반영하지 못한다. 즉, 캐시에서 유저에게 반환되는 정보가 오래된 정보일 가능성이 있다는 것이다. 이러한 단점을 보완하기 위해서 conditional GET이라는 해결책이 나왔다. 캐시가 server에 자기가 가진 정보에 날짜를 붙여서 만약 그 날짜 이후에 업데이트가 이루어졌다면 캐시쪽에서 받아오는 것이다.  

### HTTP/2  
HTTP version 1까지는 모두 ASCII format으로 사람이 보기에 적합했다. 그러나 기계입장에서는 정보의 길이가 길어질 뿐 효율적이지 못한 데이터 표현 방식이었다. 그래서 HTTP version 2에서는 binary format으로 바뀌었다. 
또한 persistent + pipelining connection의 문제였던 순차적으로 response를 보내야 하는 것도 해결되었다. 이 문제는 이전 버전에서는 reponse header에 어떤 request에 대한 응답인지 정보가 없었기애 반드시 순차적으로 보내야 해서 발생한 것인데, 이제는 stream number를 가지고 대응하는 request에 각각 응답이 준비되는대로 보내주면 된다. 더하여 각 stream에 우선순위를 줄 수 있다.  