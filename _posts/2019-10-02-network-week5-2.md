---
layout: post
title: "Network : CH 2-4"
tags: [한동대, 공부, 네트워크]
comments: true
---

> 2.4 DNS  

### DNS : Domain Name System  
사람이 binary 32 bit으로 이루어진 모든 IP주소를 외울 수 없으니 쉽게 알아보고 외울 수 있는 이름을 주소마다 붙여주었다. human-use를 위해 문자열로 되어있다 (e.g. www.google.com) 그렇다면 IP 주소와 이름은 어떻게 맵핑될까? 주소-이름/이름-주소를 맵핑하는 것을 name-address resolution 이라고 한다.  

1. Static Mapping : hostname to address mapping file or hosts file.  

2. Dynamic Mapping (DNS) : too many objects for a single management center -> distributed database system (better Scalability and Maintencance!) -> partition the name sapce into a hierarchical tree  

![Center example image](https://user-images.githubusercontent.com/35067611/66013996-25e87080-e508-11e9-88f0-74b3a005d4fe.png "Center"){: .center-image}  
사진에서 root(level 0) 바로 아래 가장 위의 node들을 TLD(Top Level Domain)이라고 하며 level 127까지 총 128 level로 이루어져있다.  

domain name은 세 부분으로 나뉘어져있다 : generic domains, country domains, inverse domain  
inverse domain이란 name -> IP address가 아니라 IP addresss -> name을 찾는 경우를 위해 있다. 예를 들어 203.252.97.22의 이름이 hisnet.handong.ed u라는 걸 찾고자 할 때 사용된다 (이 부분 더 이해 필요..)  

위와같은 tree가 전세계에 하나만 있는 것 아니다. 여러 서버가 서비스하는데 인터넷에는 총 13 root server가 전세계에 퍼져있다. 그 중 .kr DNS는 한국에 대부분 있지만 해외에도 있다. .kr name server가 우리나라에서 가장 많이 사용되지만 전부 한국에만 있다면 다른 대륙에서 접근하기에 너무 느리기 때문이다.  

### Local Name Server (default name server)  
Local name server는 계층트리에 직접적으로 속하지는 않는다. host가 DNS 쿼리를 보내면 그 쿼리는 local DNS server로 전송된다. 즉 local name server는 쿼리를 계층으로 forward해주는데 마치 proxy가 작동하는 방식과 비슷하다.  

쿼리는 iteration, recursion 두 방식으로 서버에 전송된다.  

[Iteration Queries]
![Center example image](https://user-images.githubusercontent.com/35067611/66015380-4535cc80-e50d-11e9-8221-71710257bc39.png "Center"){: .center-image}  
iterative query는 "나는 너가 찾는 IP주소를 모르겠으니 얘한테 물어봐" 느낌으로 작동한다. 주로 iterative를 많이 쓴다.  

[Recursive Queries]
![Center example image](https://user-images.githubusercontent.com/35067611/66015404-639bc800-e50d-11e9-8b54-ac0340fb9d0f.png "Center"){: .center-image}  
recursive query는 root DNS server에 상당한 부담이 간다 (heavy load at upper levels of hierarchy)  

### DNS Caching and Update Recoreds  
그런데 위에서 어떤 방식을 택하든 매번 이렇게 물어보면 Root DNS는 너무 피곤할 것이다. 중복된 것에 대해서는 물어보는 것을 생략하기 위해서 caching을 한다.  

어떤 name server든 mapping하는 것을 배우면(학습하면?) 그 서버는 mapping 을 cache한다.  
cache entries는 일정시간(TTL : time to leave)이 지난 후에는 사라진다. TLD 서버는 local name server에 캐시되어있는것이 일반적인데 이는 root name server가 자주 방문되지 않도록 하기 위함이다.  

또한 cache entries는 outdated될 수 있는데 이러 update문제는 TTL을 걸어두고 그 시간이 지나면 날려버리는 방식으로 방지한다. 그러나 이 방식으로도 여전히 완벽하진 않은데 TTL이 다 지나기전에 update되면 cache 되어있는 서버는 outdated 일 것이기 때문이다. 그래서 server update가 있다면 알려주는 기능까지 추가하여 사용한다.  

### Services Provided by DNS  
DNS는 well-known port 53번을 이용해 UDP나 TCP를 쓸 수 있다. 하지만 주로 UDP를 사용한다. 중간에 에러가 나면 안되는데 왜 UDP를 쓸까?  

TCP는 connection oriented protocol이다. connection을 만들 때, release할 때 three hand shake 과정이 필요하다. 그런데 DNS service는 주로 request-response가 거의 한 번 뿐이다. 이런 서비스에 패킷을 세개씩이나 쓰는 것은 overhead이다. 그래서 UDP를 사용하고 에러 발생 시 또 보내는 방식을 주로 택한다.  