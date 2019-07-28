---
layout: post
title: "Software Testing Study CH 3-2"
tags: [Software Testing, Fuzzing, 공부]
comments: true
---

> Efficient Grammar Fuzzing  

이전 장에서 grammr를 이용해서 어떻게 효과적이고 효율적으로 테스팅을 하는지 공부했다. 이번 장에서는 기존의 string-based algorithm을 더 빠르고 fuzz input 생성에 있어서 더 제어하기 좋은 tree-based algorithm로 바꾸어 본다.  

### An Insufficient Algorithm  
simple grammar fuzzer는 문법적으로 valid한 input을 생성했었다. 그러나 이것만으로는 충분치 않다. 말그대로 simple fuzzer는 여전히 한계가 있다. 예를 들어, 
~~~python
expr_grammar['<factor>']
~~~
~~~
['<sign-1><factor>', '(<expr>)', '<integer><symbol-1>']
~~~
위의 코드와 결과에서 expr을 제외한 모든 선택의 경우들은 symbol의 수를 증가시킨다. 우리는 symbol expansion에 maximum값을 한계로 두었었다. 그러므로 <factor>의 선택권은 (<expr>) 하나 밖에 남지 않는다. 이는 무한하게 괄호를 증가시킨다. (infinite expansion)  
이것 말고도 한계점은 더 있다. 첫번째로 비효율적이다. 매 반복에서 fuzzer는 symbol expansion을 위해 그때까지 생성된 문자열들을 모두 찾는다. 이런 알고리즘은 문자열이 자랄수록 매우 비효율적일 것이다. 두번째로는 control하기 쉽지 않다는 점이다. symbol의 갯수를 제한시킨다고 해도 여전히 매우 긴 문자열을 받을 수 있다.  

### Derivation Trees  
위으 두 단점(비효율성, control)을 극복하기 위해서 특별한 문자열 representation을 사용할 것이다. 기본 아이디어는 tree structure를 이용하는 것이다. 이런 구조는 우리로 하여금 항상 expansion status의 정보를 tracking 가능하도록 해준다. 즉, 어떤 symbol이 어떻게 확장되어왔고, 더 확장되어야 하는지 모두 알려준다. 또한, tree에 새로운 노드를 붙여나가는 것이 문자열을 매번 대체시키는 것 보다 훨씬 효율적이다.  
tree expansion을 하기 위해서는 먼저 tree traverse를 하며 자식이 없는 nonterminal symbol을 찾는다. 이를 S라 할 때 S는 더 확장되어야 하는 symbol 이다. 그리고 문법에 따라 expansion을 선택한다. 마지막으로 expansion을 S의 새로운 자식노드로 추가한다. 이 과정을 더 이상 확장할 symbol이 없을때 까지 반복한다.  

![Center example image](https://user-images.githubusercontent.com/35067611/62006019-4138a780-b176-11e9-9395-d2aa5ac4dacb.png "Center"){: .center-image}  
![Center example image](https://user-images.githubusercontent.com/35067611/62006026-544b7780-b176-11e9-837a-f423fc8a921a.png "Center"){: .center-image}  

위 사진에서 주목할 점은 string based algorithm과 다르게 tree structure는 모든 expansion trace (production history, or derivation history)를 기록하고 있다는 점이다.  

### Representing Derivation Trees  
파이썬에서 Derivation tree를 표현할 때 기본적으로 pair (SYMBOL_NAME, CHILDREN)로 표현한다. 이 때, children은 다음의 두 경우를 포함한다 :  
1. None as a placeholder for future expansion. This means that the node is a nonterminal symbol that should be expanded further.  
2. [] (i.e., the empty list) to indicate no children. This means that the node is a terminal symbol that can no longer be expanded.  

### Expanding a Node  
이제 unexpanded symbol이 들어올 때의 알고리즘을 짜보자. 전과 같이 fuzzer class를 만들고, 이 클래스가 grammar와 start symbol을 받아서 expansion을 시작할 것이다.  
~~~python
class GrammarFuzzer(Fuzzer):
    def __init__(self, grammar, start_symbol=START_SYMBOL,
                 min_nonterminals=0, max_nonterminals=10, disp=False, log=False):
        """Produce strings from `grammar`, starting with `start_symbol`.
        If `min_nonterminals` or `max_nonterminals` is given, use them as limits 
        for the number of nonterminals produced.  
        If `disp` is set, display the intermediate derivation trees.
        If `log` is set, show intermediate steps as text on standard output."""

        self.grammar = grammar
        self.start_symbol = start_symbol
        self.min_nonterminals = min_nonterminals
        self.max_nonterminals = max_nonterminals
        self.disp = disp
        self.log = log
        self.check_grammar()
    
     def new_method(self, args):
        pass

    def check_grammar(self):
        # checks the given grammar for the consistency
        assert self.start_symbol in self.grammar
        assert is_valid_grammar(
            self.grammar,
            start_symbol=self.start_symbol,
            supported_opts=self.supported_opts())

    def supported_opts(self):
        return set()
    
    def init_tree(self):
        # constructs tree w/ just the start symbol
        return (self.start_symbol, None)
    
    def expansion_to_children(expansion):
    # print("Converting " + repr(expansion))
    # strings contains all substrings -- both terminals and nonterminals such
    # that ''.join(strings) == expansion

    expansion = exp_string(expansion)
    assert isinstance(expansion, str)

    if expansion == "":  # Special case: epsilon expansion
        return [("", [])]

    strings = re.split(RE_NONTERMINAL, expansion)
    return [(s, None) if is_nonterminal(s) else (s, [])
            for s in strings if len(s) > 0]
~~~


### 질문  
- display tree function implementation  
