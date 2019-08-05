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

input reducing 을 이용해서 나온 결과를 분석하면 어떤 경우에 실패했는지 알 수 있다. 숫자가 괄호로 둘러쌓인 입력에 대해서 FAIL을 보여주고 있다. delta debuggin은 29 단계를 거쳐 이를 판단했다.  

reduced test case의 장점은 다음과 같다 :  
1. 프로그래머가 인지해야할 양도 축소시켜준다. 테스트 케이스가 더 짧고 집중될수록 버그와 상관없는 디테일에서 오는 스트레스를 받지 않아도 되는 것이다. 일반적으로 reduced input은 shorter executions, smaller program states를 야기하는데 이는 버그를 이해하는데 있어서 search space의 범위도 줄여주기에 매우 도움이 된다.  

2. 소통하기에 더 쉽다. 결국 위의 예제에서 fail에 필요한 것은 온갖 문자들의 나열이 아닌 '()' 괄호문자였다.  

3. 중복을 찾는 데 용이하다. 만약 비슷한 버그가 이미 발견, 보고되었고 축소되었다면 찾아낸 여러 다른 버그들이더라도 하나의 같은 원인을 갖고 있을 것이다. 그러므로 하나의 코드만 고치면 한번에 해결 가능하다.  

### Grammar-Based Input Reduction  
만약 input language가 문법적으로 복잡하다면 delta debugging은 여러 시도를 해보다 아예 reducing을 못 할 수도 있다. 그런 경우를 해결하기 위해 Grammar-Based Reduction(GRABR)을 소개한다. GRABR은 문법적으로 복잡한 입력을 reduce하기 위해서 grammar를 사용한다.  

### Lexical Reduction vs. Syntactic Rules  
그 견고함에도 불구하고 delta debugging이 비효율적이거나 아예 실패하는 상황이 있다. 예를 들어 1 + (2 * 3)이라는 arithmetic expression이 입력으로 들어왔다고 해보자.  

~~~
Test #1 '1 + (2 * 3)' 11 FAIL
Test #2 '2 * 3)' 6 PASS
Test #3 '1 + (' 5 PASS
Test #4 '+ (2 * 3)' 9 FAIL
Test #5 '+ ( 3)' 6 FAIL
Test #6 ' 3)' 3 PASS
Test #7 '+ (' 3 PASS
Test #8 ' ( 3)' 5 FAIL
Test #9 '( 3)' 4 FAIL
Test #10 '3)' 2 PASS
Test #11 '( ' 2 PASS
Test #12 '(3)' 3 FAIL
Test #13 '()' 2 FAIL
Test #14 ')' 1 PASS
Test #15 '(' 1 PASS
'()'
~~~

위의 결과를 보면 문법적으로 유효한 입력은 매우 소수이다. 실제로 이런 알고리즘을 적용하기 위해서는 먼저 expression이 파싱되어야 한다. 그래서 모든 invalid input을 없애고 그 후에야 위의 테스팅을 돌리는 것이 적절하다. 그렇지 않으면 delta debuggin은 완전히 실패이다. reduction중 어느것도 문법적으로 유효하지 않기 때문이며 결국 input은 복잡한 기존의 상태 그대로 남아있게 된다.  
이렇게 하기 위해서는 프로그램 자체가 input validity를 고려한 상태로 돌아가야 한다. delta debuggin은 이런 제약조건들에 대해서 이해하고 있지 않기 때문에 계속해서 validity violation을 일으킬 것이다.  

### A Grammmar-Based Reduction Approach  
문법적으로 복잡한 input을 reduce하기 위해서 string-based보다 tree로 자료구조를 바꾸어 생각해보자. 일반적인 아이디어는 input을 파싱하여 derivation tree로 만들고 시작하는 것이다. 그리고 subtree를 더 작은 subtree(same type)로 대체하는 것이다. 대안이 되는 더 작은 subtree는 다음의 둘 중 하나의 경우다.  
1. From the tree itself, or  
2. By applying an alternate grammar expansion using elements from the tree.  

### Simplifying by Replacing Subtrees  
subtree를 다른 것으로 치환하는 것은 문법의 개별적 요소가 tree에 여러번 나올 경우에만 가능하다. 아래의 예시에서는 가장 위에있는 expr를 right child expr로 대체한 경우다.  

![Center example image](https://user-images.githubusercontent.com/35067611/62441775-6b9aee00-b790-11e9-85a5-cb2c80e32e16.png "Center"){: .center-image}  

![Center example image](https://user-images.githubusercontent.com/35067611/62409288-b3463c00-b60f-11e9-85bc-1cd8aac672d1.png "Center"){: .center-image}  

### Simplifying by Alternative Expansions  
tree를 이용한 simplify 두 번째 방법은 alternative expansion이다. 위의 replacing subtree 방법과는 다르게 문법 확장을 할 때 더 작은 수의 자식을 갖는 확장이 있는지 먼저 찾고 있다면 그것으로 대체한다. 예를 들어, 아래의 경우 factor하나로 expansion하는 것을 선택한다.  
~~~
<term> ::= <term> * <factor>
~~~
~~~
<term> ::= <factor>
~~~

derivation subtree를 더 작은 subtree로 치환하고 alternate expansion을 찾아서 다시 smaller subtree 정보를 안다면 (즉, 위의 두 방법을 합친다면) 더 효과적으로 문법이 복잡한 input을 축소시킬 수 있다. 이는 delta debuggin 보다 훨씬 빠르고 (당연히) valid input만을 생성할 것이다. 하지만 이 simplification rule을 언제 적용시켜야 할지 전략이 필요하다. 다음 섹션에서는 그 전략을 어떻게 발전시킬지 알아본다.  

### Simplification Strategies  
Finding Subtrees  
대체할 subtree를 찾는 것은 간단하다. 먼저 주어진 symbol과 같은 root를 가지는 tree를 모두 반환한다.  
~~~python
def subtrees_with_symbol(self, tree, symbol, depth=-1, ignore_root=True):
    # Find all subtrees in TREE whose root is SYMBOL.
    # If IGNORE_ROOT is true, ignore the root note of TREE.

    ret = []
    (child_symbol, children) = tree
    if depth <= 0 and not ignore_root and child_symbol == symbol:
        ret.append(tree)

    # Search across all children
    if depth != 0 and children is not None:
        for c in children:
            ret += self.subtrees_with_symbol(c,
                                                symbol,
                                                depth=depth - 1,
                                                ignore_root=False)

    return ret
~~~

Alternate Expansions  
이 알고리즘은 약간 더 복잡하다. 먼저 가능한 expansion을 모두 찾는다. 그리고 각 expansion에 대해서 위의 finding subtree를 이용하여 모든 가능한 value를 채워넣는다. 그리고 구 중 첫번재 가능한 조합을 반환한다.  
~~~python
def alternate_reductions(self, tree, symbol, depth=-1):
    reductions = []

    expansions = self.grammar.get(symbol, [])
    expansions.sort(
        key=lambda expansion: len(
            expansion_to_children(expansion)))

    for expansion in expansions:
        expansion_children = expansion_to_children(expansion)

        match = True
        new_children_reductions = []
        for (alt_symbol, _) in expansion_children:
            child_reductions = self.subtrees_with_symbol(
                tree, alt_symbol, depth=depth)
            if len(child_reductions) == 0:
                match = False   # Child not found; cannot apply rule
                break

            new_children_reductions.append(child_reductions)

        if not match:
            continue  # Try next alternative

        # Use the first suitable combination
        for new_children in possible_combinations(new_children_reductions):
            new_tree = (symbol, new_children)
            if number_of_nodes(new_tree) < number_of_nodes(tree):
                reductions.append(new_tree)
                if not self.try_all_combinations:
                    break

    # Sort by number of nodes
    reductions.sort(key=number_of_nodes)

    return reductions
~~~

Both Strategies Together  
둘을 합쳐보자. 먼저 존재하는 subtree를 찾고, alternate expansion을 선택한다.  
~~~python
class GrammarReducer(GrammarReducer):
    def symbol_reductions(self, tree, symbol, depth=-1):
        """Find all expansion alternatives for the given symbol"""
        reductions = (self.subtrees_with_symbol(tree, symbol, depth=depth)
                      + self.alternate_reductions(tree, symbol, depth=depth))

        # Filter duplicates
        unique_reductions = []
        for r in reductions:
            if r not in unique_reductions:
                unique_reductions.append(r)

        return unique_reductions
~~~

### A Depth-Oriented Strategy  
지금도 충분히 좋은 결과를 내는 알고리즘이지만 더 좋게 만들 수 있다. 대체할 때 subtree의 사이즈가 더 작으면 더 효율적으로 프로그램을 진행할 수 있지만 시간이 꽤 오래 걸릴 것이다. 그래서 처음에는 large subtree에 대해서만 고려해보자. 이를 위해 depth를 대체가능한 subtree를 찾을수 있는 정도 까지만 제한한다. 먼저 direct descendant를 보고, 그 다음에 더 아래에 있는 자녀들을 본다.  

~~~python
def reduce_tree(self, tree):
    depth = 0
    while depth < max_height(tree):
        reduced = self.reduce_subtree(tree, tree, depth)
        if reduced:
            depth = 0    # Start with new tree
        else:
            depth += 1   # Extend search for subtrees
    return tree 
~~~
depth를 0부터 시작해서 진행될 수록 증가시키는 것이 이 알고리즘의 핵심이다.  

### 배운 점  
- failure-inducing input을 reduce하는 것은 테스팅과 디버깅에 매우 효율적이다.  
- delta debuggin은 간단하지만 좋은 알고리즘이다.  
- 하지만 syntactically compelx inmput에 대해서는 grammar-based reduction이 더 효과적이다.  