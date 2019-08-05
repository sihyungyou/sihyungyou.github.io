---
layout: post
title: "Software Testing Study CH 4-1"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Mining Input Grammars  

and write a grammar in the first place. While the grammars we have seen so far have been rather simple, creating a grammar for complex inputs can involve quite some effort. In this chapter, we therefore introduce techniques that automatically mine grammars from programs – by executing the programs and observing how they process which parts of the input. In conjunction with a grammar fuzzer, this allows us to
1. take a program,
2. extract its input grammar, and
3. fuzz it with high efficiency and effectiveness, using the concepts in this book.

### A Grammar Challenge  
We found from the chapter on parsers that coarse grammars do not work well for fuzzing when the input format includes details expressed only in code. That is, even though we have the formal specification of CSV files (RFC 4180), the inventory system includes further rules as to what is expected at each index of the CSV file. The solution of simply recombining existing inputs, while practical, is incomplete. In particular, it relies on a formal input specification being available in the first place. However, we have no assurance that the program obeys the input specification given.  

ne of the ways out of this predicament is to interrogate the program under test as to what its input specification is. That is, if the program under test is written in a style such that specific methods are responsible for handling specific parts of the input, one can recover the parse tree by observing the process of parsing. Further, one can recover a reasonable approximation of the grammar by abstraction from multiple input trees.  

We start with the assumption (1) that the program is written in such a fashion that specific methods are responsible for parsing specific fragments of the program -- This includes almost all ad hoc parsers.

The idea is as follows:

Hook into the Python execution and observe the fragments of input string as they are produced and named in different methods.
Stitch the input fragments together in a tree structure to retrieve the Parse Tree.
Abstract common elements from multiple parse trees to produce the Context Free Grammar of the input.  

### A Simple Grammar Miner  
