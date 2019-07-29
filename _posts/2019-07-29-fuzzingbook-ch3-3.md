---
layout: post
title: "Software Testing Study CH 3-3"
tags: [Software Testing, Fuzzing, 공부]
comments: true
---

> Grammar Coverage  

Grammar에 기초하여 input을 생성시키는 것은 모든 가능한 문법적 확장을 할 수 있도록 해준다. 그러나 포괄적인 test suite를 만드는 것은 최대한 많은 변화를 갖는 것이 좋다. 예를 들어 같은 expansion을 반복하는 것은 하지 않는 것이다. 이번 장에서는 어떻게 시스템적으로 grammar에 있는 모든 요소들을 cover 하는지에 대해서 공부한다.  

### Covering Grammar Elements  
test suite generation의 목표는 프로그램의 모든 기능을 cover하는 것이다. 물론 이는 기능의 실패 또한 포함한다. 하지만 기능이라는 것은 input structure에 매우 의존적이다. 특정한 입력이 들어오지 않는다면 그와 관련된 코드 혹은 기능은 아예 실행되지 않을 것이고 그런 영역에서 버그를 찾을 기회도 없어지는 것이다. 예를 들어 이전 장에서 expr_grammar을 가지고 테스팅을 할 때 실수나 음수에 대해서 test suite이 만들어지지 않았더라면 실수와 음수에 대해서는 기능적 테스트가 이루어지지 않았을 것이다. 그러므로 우리의 목표는 모든 가능한 문법적 확장에 대해 cover하는 것이다.  
다양함을 최대치로 끌어올리는데 좋은 한 가지 방법은 grammar production 중에 expansion을 tracking하는 것이다. 이미 cover한 expansion 보다는 가능한 다른 expansion candidate을 고를 수 있기 때문이다.

~~~
['+<factor>', '-<factor>', '(<expr>)', '<integer>.<integer>', '<integer>']
~~~
예를 들어 factor의 첫번째 expansion으로 이미 integer를 선택했다면 (그리고 그 정보를 가지고 있다면) integer는 "already covered"로 표시하고 다른 요소들로 넘어갈 수 있는 것이다.

### Tracking Grammar Coverage  
GrammarCoverageFuzzer class는 현재까지 커버된 문법들을 tracking한다.  
~~~python
def _max_expansion_coverage(self, symbol, max_depth):
    if max_depth <= 0:
        return set()

    self._symbols_seen.add(symbol)

    expansions = set()
    for expansion in self.grammar[symbol]:
        expansions.add(expansion_key(symbol, expansion))
        for nonterminal in nonterminals(expansion):
            if nonterminal not in self._symbols_seen:
                expansions |= self._max_expansion_coverage(
                    nonterminal, max_depth - 1)

    return expansions
~~~
_max_expansion_coverage는 start symbol로 부터 시작해서 recursive하게 expansion을 수행하고 그 정보를 저장해둔다. max expansion coverage 에서 expansion coverage를 빼면 missing coverage를 쉽게 구할 수 있다.  

### Covering Grammar Expansions  
이제 다음 세 아이디어를 가지고 coverage를 만들어(?) 보자.  
1. We determine children yet uncovered (in uncovered_children)  
2. If all children are covered, we fall back to the original method (i.e., choosing one expansion randomly)  
3. Otherwise, we select a child from the uncovered children and mark it as covered.  
~~~python
class SimpleGrammarCoverageFuzzer(TrackingGrammarCoverageFuzzer):
    def choose_node_expansion(self, node, possible_children):
        # Prefer uncovered expansions
        (symbol, children) = node
        uncovered_children = [c for (i, c) in enumerate(possible_children)
                              if expansion_key(symbol, c) not in self.covered_expansions]
        index_map = [i for (i, c) in enumerate(possible_children)
                     if c in uncovered_children]

        if len(uncovered_children) == 0:
            # All expansions covered - use superclass method
            return self.choose_covered_node_expansion(node, possible_children)

        # Select from uncovered nodes
        index = self.choose_uncovered_node_expansion(node, uncovered_children)

        return index_map[index]
~~~

~~~python
for i in range(7):
    print(f.fuzz(), end=" ")
~~~
결과
~~~
0 9 7 4 8 3 6 
~~~
fuzzing 할 때마다 새로운 (preferring uncovered expansion) expansion을 하는 것을 볼 수 있다.  

### Deep Foresight  
하지만 이런 방법으로 모든 복잡한 grammar에 적용하기에는 부족하다. 예를 들어 CGI Grammar에 대해서 위와 같은 방식으로 fuzzing을 하면 uncovered coverage가 여전히 많이 남아있기 때문이다. 이는 특정 symbol에 대해서 maximum set of expansions reachable을 고려하지 않았기 때문이다. (물론 이 알고리즘은 max expansion 함수에 구현은 되어있다) 이 max expansion 함수를 child단위에 적용시키는 과정이 필요하다.  
~~~python
class GrammarCoverageFuzzer(SimpleGrammarCoverageFuzzer):
    def _new_child_coverage(self, children, max_depth):
        new_cov = set()
        for (c_symbol, _) in children:
            if c_symbol in self.grammar:
                new_cov |= self.max_expansion_coverage(
                    c_symbol, max_depth)
        return new_cov

    def new_child_coverage(self, symbol, children, max_depth=float('inf')):
        """Return new coverage that would be obtained by expanding (symbol, children)"""
        new_cov = self._new_child_coverage(children, max_depth)
        new_cov.add(expansion_key(symbol, children))
        new_cov -= self.expansion_coverage()   # -= is set subtraction
        return new_cov
~~~

이제부터는 expansion을 고를 때 new_child_coverage() 함수를 이용해서 가장 많은 새로운(unseen) coverage를 제공할 child를 고를 수 있게 되었다.  

### Adaptive Lookahead  
child를 선택할 때 maximum overall coverage to be obtained 를 확인하지 않는다면 많은 uncovered expansion이 남을 것이다. 그래서 BFS 방식으로 하는데, 먼저 모든 expansion에 대해 주어진 depth까지 cover한다. 그리고 greater depth만 찾아다니는 알고리즘이다. maximum depth는 0부터 시작해서 uncovered expansion을 찾을 떄 까지 계속 증가한다.  
### All Together  
종합해보면, 먼저 얻을 수 있는 possible coverages를 결정한다. 그리고 maximum coverage를 주는 child들 중에서 random으로 선택해서 fuzzing한다.  