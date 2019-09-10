---
layout: post
title: "Network : Intro 04"
tags: [한동대, 공부, 네트워크]
comments: true
---

> 1.4 Performance : Delay, loss throughput  

### Performance in Networks  
네트워크에서 퍼포먼스란 크게 throughput(초당 얼마나 많은 data를 전송하는가), delay(time), loss(손실)의 측면으로 본다. Throughput은 일반적으로 bit per second (bps)로 표현된다. Delay는 transmission time + propagation delay + queueing delay + processing delay를 모두 합한 시간을 칭한다.  

### Four sources of Packet delay  
1. Processing delay  
checking bit errors : 에러가 발생하는지 체크한다.  
determine output link : forwarding table에서 찾아서 결정한다.  
위의 두 경우는 전자는 H/W에서, 후자는 small table안에서 이루어지기 때문에 무시해도 될 정도로 빠르다.  

2. Queueing delay  
Waiting time in a queue for transmission.  
Queueing delay는 variation이 매우 크다. 이는 router의 congestion level에 따라 달라지기 때문이다. 이러한 delay variation을 "jitter"라고 한다.  
Output buffer (queue)의 딜레마는 버퍼의 사이즈가 너무 작으면 data loss가 커지고 그렇다고 너무 크면 delay가 높아지기 때문에 적절한 size를 찾는 것이다.  

3. Transmission delay  
Packet을 전송할 때 걸리는 시간으로 보통 R = link bandwidth(bps), L = packet length(bits) 라고 할 때 transmission delay는 L/R이다.  

4. Propagation delay  
bit이 source로부터 destination까지 전달되는 데에 걸리는 시간으로 d = length of physical link (실제 물리적 거리), s = propagation speed in medium (보통 자유 공간에서 빛의 속도이지만 광섬유 같은 매체를 통해서 가는 속도) 라고 할 때 propagation delay는 d/s이다.  

### Delay in Packet-Switched Networks  
![Center example image](https://user-images.githubusercontent.com/35067611/64605940-180b6800-d400-11e9-8401-a757a1abe2cd.png "Center"){: .center-image}  

위 그림과 같이 중간에 router가 없고 source-destination 거리가 4800km 경우의 예시다. 보내야 할 데이터의 양은 1Mbit, link는 64kbps이다. link의 신호의 속도는 2*10^8 m/s이며 processing delay는 무시한다.  

Packet-switching이기 때문에 call setup time은 필요 없다. 즉, total delay = propagation delay + transmission delay로 표현할 수 있을 것이다.  

propagation delay : d/s (즉, 4800km 거리를 link의 신호 속도로 나눈 시간)이므로 24 msec  
transmission delay : 1M bit / 64kbps (즉, 보내야할 데이터의 양을 link의 bps로 나눈 시간)이므로 15.625 sec  
이 때 두 delay의 단위를 보면 prop delay는 msec, trans delay는 sec로 전자가 무시될 수 있을 만큼 작다.  

하지만 같은 예제에 R이 64 kbps가 아니라 훨씬 빠른 1 Gbps라면 이야기가 달라진다. prop delay는 24msec으로 같겠으나, trans delay가 1ms로 줄어들어 결코 prop delay를 무시할 수 없는 것이다.  

### Delay Comparison  
위에서 Packet-Switched Networks에서도 link의 속도가 달라지면 delay 차이가 매우 심하다는 것을 알았다. 그렇다면 여러 다른 네트워크 방식끼리 delay를 비교해보면 어떨까? 조건은 다음과 같다 :  
number of hops (N) : 3  
message length (L) : 5500 bits  
link rate (R) : 9600 bps  
packet size (payload + header) (P) : 1040 bits  
header overload (H) : 40 bits  
circuit establishment time : 0.2 sec  
prop delay per hop : 1 ms  
(queueing, processing delay는 무시한다)  

![Center example image](https://user-images.githubusercontent.com/35067611/64606563-7ab13380-d401-11e9-91b3-926b9ce53daa.png "Center"){: .center-image}  

![Center example image](https://user-images.githubusercontent.com/35067611/64606569-7edd5100-d401-11e9-89ce-41f9d61505cf.png "Center"){: .center-image}  

![Center example image](https://user-images.githubusercontent.com/35067611/64606570-7edd5100-d401-11e9-87e1-2beef5b61060.png "Center"){: .center-image}  

이렇게 어떤 네트워크 설계를 이용하느냐에 따라서 delay를 계산하는 방식도 달라지고 결과도 당연히 다르다.  

### Real Internet Delays  
linux 명령어 중 traceroute 라는 것이 있다. 이는 유저가 주는 destination 까지의 delay를 계산하여 로그를 찍어준다.  
이것이 작동하는 방식은 TTL(time to live, 생명이 남은 시간)을 초기에 설정한 후 router를 지날 떄 마다 1씩 감소시키고 만약 destination에 도착하지 못했는데 TTL = 0 이면 그 path에는 문제가 있으므로 source에게 notice하고, packet을 버리는 방식이다. 이런 방식을 이용해 router 갯수를 고려한 TTL이 설정된 probe를 보내 source에게 도착했다고 notice되는 데 까지 걸리는 시간을 delay로 간주하여 계산하는 것이다.  
![Center example image](https://user-images.githubusercontent.com/35067611/64606982-818c7600-d402-11e9-8978-cff1e36bfc62.png "Center"){: .center-image}  

### Throughput  
throughput이란 bits/time unit 단위의 source-destination사이에 전송할 수 있는 순간의 rate이다. 여러 link에 걸쳐있는 path라면 Rs bits/sec, Rc bits/sec, Ra bits/sec 등 여러 다른 rate이 존재할 것이다. 이 때 어 성능(rate)이 좋은 link가 fat pipe, 그렇지 않은 link는 thin pipe이다. 이렇게 속도가 다른 여러 link가 있을 때 average end-end throughput은 더 작은 (혹은 가장 작은) rate에 맞춰진다. 이런 속도가 더 낮은 link는 항상 존재할 것이고 이것에 의해 throughput이 결정된다. 그리고 그 link(즉, link on end-end path that constrains end-end throughput)를 bottlenek link라고 한다.  