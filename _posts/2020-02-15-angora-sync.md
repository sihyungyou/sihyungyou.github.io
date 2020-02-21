---
layout: post
title: "Angora Fuzzer Distribution & Parallelization"
tags: [Fuzzing, Angora, 캡스톤]
comments: true
---

> 분산환경에서의 앙고라 뻐저  

Fuzzing tool을 돌리는 것은 컴퓨팅 파워가 소모되는 작업이다. 한 타겟 프로그램의 버그 검출을 위해 fuzzing으로 테스팅을 할 때 하나의 컴퓨터가 아니라 여러대의 컴퓨터로 작업을 돌리면 테스팅 결과의 질이 반드시 향상된다고 보장할 수는 없으나 효과적일 것으로 기대된다. 이런 병렬화 작업에서 요점은 첫째, 분산환경 구축 및 master가 slave 일을 나누어주는 통신과정 둘째, 각 slave 서버들의 결과를 실시간으로 sync를 맞추어서 더 효과적인 fuzzing performance를 내는 것이다.  

전자는 쉽게 생각하면 여러 TCP Connection이 이루어져서 target program과 seed 등 fuzzing에 필요한 파일들을 넘기는 FTP 작업이다. 후자는 fuzzer의 internal state에 대한 정의가 확실하게 내려지고 그것들을 어떻게 주고받을 것인지, seed 배분은 어떻게 할 것인지 등 논의할 사항이 많아 아직 개선하는 과정에 있다.  

![Center example image](https://user-images.githubusercontent.com/35067611/75041542-1d68f480-5500-11ea-9d77-822b90cb71fe.gif "Center"){: .center-image}  