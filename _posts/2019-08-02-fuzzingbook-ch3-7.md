---
layout: post
title: "Software Testing Study CH 3-7"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Greybox Fuzzing with Grammars  

In this chapter, we introduce two important extensions to our syntactic fuzzing techniques:

We show how to combine parsing and fuzzing with grammars. This allows to mutate existing inputs while preserving syntactical correctness, and to reuse fragments from existing inputs while generating new ones. The combination of parsing and fuzzing, as demonstrated in this chapter, has been highly successful in practice: The LangFuzz fuzzer for JavaScript has found more than 2,600 bugs in JavaScript interpreters this way.

In the previous chapters, we have used grammars in a black-box manner – that is, we have used them to generate inputs regardless of the program being tested. In this chapter, we introduce mutational greybox fuzzing with grammars: Techniques that make use of feedback from the program under test to guide test generations towards specific goals. As in lexical greybox fuzzing, this feedback is mostly coverage, allowing us to direct grammar-based testing towards uncovered code parts.

### Background  
First, we recall a few basic ingredients for mutational fuzzers.

Seed. A seed is an input that is used by the fuzzer to generate new inputs by applying a sequence of mutations.
Mutator. A mutator implements a set of mutation operators that applied to an input produce a slightly modified input.
PowerSchedule. A power schedule assigns energy to a seed. A seed with higher energy is fuzzed more often throughout the fuzzing campaign.
MutationFuzzer. Our mutational blackbox fuzzer generates inputs by mutating seeds in an initial population of inputs.
GreyboxFuzzer. Our greybox fuzzer dynamically adds inputs to the population of seeds that increased coverage.
FunctionCoverageRunner. Our function coverage runner collects coverage information for the execution of a given Python function.

### Building a Keyword Dictionary  
To fuzz our HTML parser, it may be useful to inform a mutational fuzzer about important keywords in the input – that is, important HTML keywords. To this end, we extend our mutator to consider keywords from a dictionary.
~~~python
dict_mutator = DictMutator(["<a>","</a>","<a/>", "='a'"])
~~~

![Center example image](https://user-images.githubusercontent.com/35067611/62351047-70b52e80-b53f-11e9-9cc3-c32ac8a15e3c.png "Center"){: .center-image}  

Informing the fuzzer about important keywords already goes a long way towards achieving lots of coverage quickly.

### Fuzzing with Input Fragments  
