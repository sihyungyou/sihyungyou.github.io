---
layout: post
title: "Network : Intro 04"
tags: [한동대, 공부]
comments: true
---

> 1.4 Performance : Delay, loss throughput  

### Performance in Networks  
네트워크에서 퍼포먼스란 크게 throughput(초당 얼마나 많은 data를 전송하는가), delay(time), loss(손실)의 측면으로 본다. Throughput은 일반적으로 bit per second (bps)로 표현된다. Delay는 transmission time + propagation delay + queueing delay + processing delay를 모두 합한 시간을 칭한다.  

### Four sources of Packet delay  
#### Processing delay  
- checking bit errors : 에러가 발생하는지 체크한다.  
- determine output link : forwarding table에서 찾아서 결정한다.  
위의 두 경우는 전자는 H/W에서, 후자는 small table안에서 이루어지기 때문에 무시해도 될 정도로 빠르다.  

#### Queueing delay  
Waiting time in a queue for transmission.  
Queueing delay는 variation이 매우 크다. 이는 router의 congestion level에 따라 달라지기 때문이다. 이러한 delay variation을 "jitter"라고 한다.  
Output buffer (queue)의 딜레마는 버퍼의 사이즈가 너무 작으면 data loss가 커지고 그렇다고 너무 크면 delay가 높아지기 때문에 적절한 size를 찾는 것이다.  

