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

실행 및 결과  
~~~python
dd_reducer = DeltaDebuggingReducer(mystery, log_test=True)
dd_reducer.reduce(failing_input)
~~~
~~~
Test #1 ' 7:,>((/$$-/->.;.=;(.%!:50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#' 97 FAIL
Test #2 '\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#' 49 PASS
Test #3 " 7:,>((/$$-/->.;.=;(.%!:50#7*8=$&&=$9!%6(4=&69':" 48 PASS
Test #4 '50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#' 73 FAIL
Test #5 "50#7*8=$&&=$9!%6(4=&69':<7+1<2!4$>92+$1<(3%&5''>#" 49 PASS
Test #6 '50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+' 48 FAIL
Test #7 '\'<3+0-3.24#7=!&60)2/+";+' 24 PASS
Test #8 "50#7*8=$&&=$9!%6(4=&69':" 24 PASS
Test #9 '9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+' 36 FAIL
Test #10 '9!%6(4=&69\':=!&60)2/+";+' 24 FAIL
Test #11 '=!&60)2/+";+' 12 PASS
Test #12 "9!%6(4=&69':" 12 PASS
Test #13 '=&69\':=!&60)2/+";+' 18 PASS
Test #14 '9!%6(4=!&60)2/+";+' 18 FAIL
Test #15 '9!%6(42/+";+' 12 PASS
Test #16 '9!%6(4=!&60)' 12 FAIL
Test #17 '=!&60)' 6 PASS
Test #18 '9!%6(4' 6 PASS
Test #19 '6(4=!&60)' 9 FAIL
Test #20 '6(460)' 6 FAIL
Test #21 '60)' 3 PASS
Test #22 '6(4' 3 PASS
Test #23 '(460)' 5 FAIL
Test #24 '460)' 4 PASS
Test #25 '(0)' 3 FAIL
Test #26 '0)' 2 PASS
Test #27 '(' 1 PASS
Test #28 '()' 2 FAIL
Test #29 ')' 1 PASS
'()'
~~~

Now we know why MysteryRunner fails – it suffices that the input contains two matching parentheses with a number between them. Delta Debugging determines this is 29 steps. Its result is 1-minimal, meaning that every character contained is required to produce the error; removing any (as seen in tests #27 and #29, above) no longer makes the test fail. This property is guaranteed by the delta debugging algorithm, which in its last stage always tries to delete characters one by one.

Delta debugging의 장점은 다음과 같다 : 
A reduced test case reduces the cognitive load of the programmer. The test case is shorter and focused, and thus does not burden the programmer with irrelevant details. A reduced input typically leads to shorter executions and smaller program states, both of which reduce the search space as it comes to understanding the bug. In our case, we have eliminated lots of irrelevant input – only the two characters the reduced input contains are relevant.

A reduced test case is easier to communicate. All one needs here is the summary: MysteryRunner fails on "()", which is much better than MysteryRunner fails on a 4100-character input (attached).

A reduced test case helps in identifying duplicates. If similar bugs have been reported already, and all of them have been reduced to the same cause (namely that the input contains matching parentheses), then it becomes obvious that all these bugs are different symptoms of the same underlying cause – and would all be resolved at once with one code fix.

### Grammar-Based Input Reduction  
If the input language is syntactically complex, delta debugging may take several attempts at reduction, and may not be able to reduce inputs at all. In the second half of this chapter, we thus introduce an algorithm named Grammar-Based Reduction (or GRABR for short) that makes use of grammars to reduce syntactically complex inputs.

### Lexical Reduction vs. Syntactic Rules  
Despite its general robustness, there are situations in which delta debugging might be inefficient or outright fail. As an example, consider some expression input such as 1 + (2 * 3). Delta debugging requires a number of tests to simplify the failure-inducing input, but it eventually returns a minimal input

Looking at the tests, above, though, only few of them actually represent syntactically valid arithmetic expressions. In a practical setting, we may want to test a program which actually parses such expressions, and which would reject all invalid inputs. We define a class EvalMysteryRunner which first parses the given input (according to the rules of our expression grammar), and only if it fits would it be passed to our original MysteryRunner. This simulates a setting in which we test an expression interpreter, and in which only valid inputs can trigger the bug.

Under these circumstances, it turns out that delta debugging utterly fails. None of the reductions it applies yield a syntactically valid input, so the input as a whole remains as complex as it was before.

This behavior is possible if the program under test has several constraints regarding input validity. Delta debugging is not aware of these constraints (nor of the input structure in general), so it might violate these constraints again and again.

### A Grammmar-Based Reduction Approach  
To reduce inputs with high syntactical complexity, we use another approach: Rather than reducing the input string, we reduce the tree representing its structure. The general idea is to start with a derivation tree coming from parsing the input, and then substitute subtrees by smaller subtrees of the same type. These alternate subtrees can either come
1. From the tree itself, or
2. By applying an alternate grammar expansion using elements from the tree.

### Simplifying by Replacing Subtrees  
Replacing one subtree by another only works as long as individual elements such as expr occur multiple times in our tree. In the reduced new_derivation_tree, above, we could replace further expr trees only once more.

### Simplifying by Alternative Expansions  
A second means to simplify this tree is to apply alternative expansions. That is, for a symbol, we check whether there is an alternative expansion with a smaller number of children. Then, we replace the symbol with the alternative expansion, filling in needed symbols from the tree.
