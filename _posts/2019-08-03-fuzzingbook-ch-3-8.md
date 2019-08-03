---
layout: post
title: "Software Testing Study CH 3-8"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Reducing Failure-Inducing Inputs  

fuzzer generatred input은 사람이 읽기 힘든 형태로도 생성된다. 이런 input들은 디버깅 과정에서 정확히 어떤 이유로 프로그램 실패를 야기시켰는지 분석하기 힘들게 한다. 이번 장에서는 자동으로 failure-inducint input을 축소시키고 간단하게 만드는 기술에 대해 공부한다.  

### Why Reducing?  
지금까지 fuzzing에 대해 공부하면서 프로그램을 실패하도록 trigger하는 많은 테스트 케이스 생성 기술을 봐 왔다. 만약 그것이 성공했다면 거기서 끝날 것이 아니라 왜 실패했는지, 어떻게 고쳐야 하는지 분석해야 한다.  

~~~
' 7:,>((/$$-/->.;.=;(.%!:50#7*8=$&&=$9!%6(4=&69\':\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#'
~~~
위와 같은 입력이 프로그램 실패를 trigger 했다고 가정하면 어떤 상황과 조건에서 버그를 잡은 것인지 분석하기 매우 복잡해진다.  

### Manual Input Reduction  
디버깅 과정의 중요한 부분은 reduction이다. reduction은 찾아낸 버그와 실제로 관련이 있는 조건과 상황을 가진 input만 남겨두고 관련 없는 것들은 없애는 과정을 말한다. 즉, 모든 문제상황에 대해 관련성이 있는지 체크한다. 없다면 problem report에서 삭제한다.  
input에 대해서 divide and conquer 접근을 한다. 이진탐색을 실행하고 여전히 버그를 검출하는 반만 남겨 놓는다. 그렇지 않다면 이전으로 돌아가 input의 다른 반으로 다시 테스팅을 반복한다.  
first half
~~~
(" 7:,>((/$$-/->.;.=;(.%!:50#7*8=$&&=$9!%6(4=&69':", 'PASS')
~~~

second half
~~~
('\'<3+0-3.24#7=!&60)2/+";+<7+1<2!4$>92+$1<(3%&5\'\'>#', 'PASS')
~~~
하지만 이렇게 input을 반으로 쪼개어 테스트 했음에도 불구하고 모두 PASS인 경우가 있다. 이럴 때는 더 작은 덩어리로 나눈다. 이 과정은 문자열 입력에서 한 문자 단위까지 계속해서 쪼갠다. 하지만 이런 알고리즘은 문자열이 길어질수록 매우 비효율적이라는 단점이 있다. 효과적으로, 자동으로 failure-inducing input을 reduce 할 알고리즘이 필요하다.  

### Delta Debugging  
위에서 개선시킨 효과적인 알고리즘 중 하나는 delta debugging이다. 기본적으로 이진탐색으로 구현되어 있으나 약간의 변형을 준 알고리즘이다. 만약 위의 경우와 같이 first, second half가 모두 버그검출에 실패한다면 한 문자 단위까지 계속해서 문자열을 잘라나간다. 결국 첫 번째 반을 자른 후에는 첫 번째 쿼터, 두 번째 쿼터 .. 이렇게 계속 반복한다.  

예시)
removing first quarter : FAIL  
removing first and second quarter (test with second half) : PASS  
removing first and third quarter : PASS  
removing first and fourth quarter : FAIL  

결론적으로 첫 번째와 네 번째 quarer로만 이루어진 input으로(50%) reduce할 수 있다.  

DeltaDebuggingReducer 클래스는 다음과 같이 정의된다. n은 2부터 시작하고 버그를 찾지 못하면 n은 두 배가 되면서 문자열의 길이와 동일해 질 때 까지 다음 반복을 진행한다.  

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

If we replace derivation subtrees by (smaller) subtrees, and if we search for alternate expansions that again yield smaller subtrees, we can systematically simplify the input. This could be much faster than delta debugging, as our inputs would always be syntactically valid. However, we need a strategy for when to apply which simplification rule. This is what we develop in the remainder of this section.

### A Class for Reducing with Grammars  

