---
layout: post
title: "Fuzzingbook Exercises CH 2-3"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Mutation-Based Fuzzing  

### Exercise 2: Fuzzing bc with Mutations  
Apply the above mutation-based fuzzing technique on `bc`, as in the chapter ["Introduction to Fuzzing"](Fuzzer.ipynb).  

#### Part 1: Non-Guided Mutations  
Start with non-guided mutations.  How many of the inputs are valid?  
~~~python
from Fuzzer import ProgramRunner

seed = ["1 + 1"]
bc = ProgramRunner(program="bc")
m = MutationFuzzer(seed)
outcomes = m.runs(bc, trials=100)

outcomes[:3]
~~~

Programrunner 클래스로 bc 프로그램을 실행하고 MutationFuzzer로 seed를 mutate한 정보를 저장해둔다. 그리고 bc프로그램을 input으로 저장한 mutations들에 대해서 실행한 후, outcomes를 출력한다.  