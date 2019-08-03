---
layout: post
title: "Software Testing Study CH 3-8"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Reducing Failure-Inducing Inputs  

By construction, fuzzers create inputs that may be hard to read. This causes issues during debugging, when a human has to analyze the exact cause of the failure. In this chapter, we present techniques that automatically reduce and simplify failure-inducing inputs to a minimum in order to ease debugging.

### Why Reducing?  
At this point, we have seen a number of test generation techniques that all in some form produce inputs in order to trigger failures. If they are successful – that is, the program actually fails – we must find out why the failure occurred and how to fix it.  

~~~
' 7:,>((/$$-/->.;.=;(.%!:50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#'
~~~
위와 같은 입력이 프로그램 실패를 trigger 했다고 가정하면 어떤 상황과 조건에서 버그를 잡은 것인지 분석하기 매우 복잡해진다.  

### Manual Input Reduction  
One important step in the debugging process is reduction – that is, to identify those circumstances of a failure that are relevant for the failure to occur, and to omit (if possible) those parts that are not. As Kernighan and Pike [Kernighan et al, 1999.] put it:
> One important step in the debugging process is reduction – that is, to identify those circumstances of a failure that are relevant for the failure to occur, and to omit (if possible) those parts that are not. As Kernighan and Pike [Kernighan et al, 1999.] put it:

Specifically for inputs, they suggest a divide and conquer process:
> Proceed by binary search. Throw away half the input and see if the output is still wrong; if not, go back to the previous state and discard the other half of the input.

first half
~~~
(" 7:,>((/$$-/->.;.=;(.%!:50#7*8=$&&=$9!%6(4=&69':", 'PASS')
~~~

second half
~~~
('\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#', 'PASS')
~~~
This did not go so well either. We may still proceed by cutting away smaller chunks – say, one character after another. If our test is deterministic and easily repeated, it is clear that this process eventually will yield a reduced input. But still, it is a rather inefficient process, especially for long inputs. What we need is a strategy that effectively minimizes a failure-inducing input – a strategy that can be automated.  

### Delta Debugging  
One strategy to effectively reduce failure-inducing inputs is delta debugging [Zeller et al, 2002.]. Delta Debugging implements the "binary search" strategy, as listed above, but with a twist: If neither half fails (also as above), it keeps on cutting away smaller and smaller chunks from the input, until it eliminates individual characters. Thus, after cutting away the first half, we cut away the first quarter, the second quarter, and so on.

removing first quarter : fail  
removing first and second quarter (test with second half) : pass  
removing first and third quarter : pass  
removing first and fourth quarter : fail

결론적으로 첫 번째와 네 번째 quarer로만 이루어진 input으로(50%) reduce할 수 있다.  

We have now tried to remove pieces that make up  12  and  14  of the original failing string. In the next iteration, we would go and remove even smaller pieces –  18 ,  116  and so on. We continue until we are down to  197  – that is, individual characters.

However, this is comething we happily let a computer do for us. We first introduce a Reducer class as an abstract superclass for all kinds of reducers. The test() methods runs a single test (with logging, if so wanted); the reduce() method will eventually reduce an input to the minimum.

Our implementation uses almost the same Python code as Zeller in [Zeller et al, 2002.]; the only difference is that it has been adapted to work on Python 3 and our Runner framework. The variable n (initially 2) indicates the granularity – in each step, chunks of size  1n  are cut away. If none of the test fails (some_complement_is_failing is False), then n is doubled – until it reaches the length of the input.

~~~python
class DeltaDebuggingReducer(CachingReducer):
    def reduce(self, inp):
        self.reset()
        assert self.test(inp) != Runner.PASS

        n = 2     # Initial granularity
        while len(inp) >= 2:
            start = 0
            subset_length = len(inp) / n
            some_complement_is_failing = False

            while start < len(inp):
                complement = inp[:int(start)] + \
                    inp[int(start + subset_length):]

                if self.test(complement) == Runner.FAIL:
                    inp = complement
                    n = max(n - 1, 2)
                    some_complement_is_failing = True
                    break

                start += subset_length

            if not some_complement_is_failing:
                if n == len(inp):
                    break
                n = min(n * 2, len(inp))

        return inp
~~~
