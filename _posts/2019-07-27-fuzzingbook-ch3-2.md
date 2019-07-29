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
위의 코드와 결과에서 expr을 제외한 모든 선택의 경우들은 symbol의 수를 증가시킨다. 우리는 symbol expansion에 maximum값을 한계로 두었었다. 그러므로 factor의 선택권은 (expr) 하나 밖에 남지 않는다. 이는 무한하게 괄호를 증가시킨다. (infinite expansion)  
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
        """ checks the given grammar for the consistency """
        assert self.start_symbol in self.grammar
        assert is_valid_grammar(
            self.grammar,
            start_symbol=self.start_symbol,
            supported_opts=self.supported_opts())

    def supported_opts(self):
        return set()
    
    def init_tree(self):
        """ constructs tree w/ just the start symbol """
        return (self.start_symbol, None)
    
    def expansion_to_children(expansion):
        """ takes an expansion string and decomposes it into a list of derivation trees """

        expansion = exp_string(expansion)
        assert isinstance(expansion, str)

        if expansion == "":  # Special case: epsilon expansion
            return [("", [])]

        strings = re.split(RE_NONTERMINAL, expansion)
        return [(s, None) if is_nonterminal(s) else (s, [])
                for s in strings if len(s) > 0]

    def choose_node_expansion(self, node, possible_children):
        """Return index of expansion in `possible_children` to be selected.  Defaults to random."""
        return random.randrange(0, len(possible_children))

    def expand_node_randomly(self, node):
        """ take some unexpanded node in the tree, choose a random expansion, and return the new tree """
        (symbol, children) = node
        assert children is None

        if self.log:
            print("Expanding", all_terminals(node), "randomly")

        # Fetch the possible expansions from grammar...
        expansions = self.grammar[symbol]
        possible_children = [self.expansion_to_children(
            expansion) for expansion in expansions]

        # ... and select a random expansion
        index = self.choose_node_expansion(node, possible_children)
        chosen_children = possible_children[index]

        # Process children (for subclasses)
        chosen_children = self.process_chosen_children(chosen_children,
                                                       expansions[index])

        # Return with new children
        return (symbol, chosen_children)
    
    def process_chosen_children(self, chosen_children, expansion):
        """Process children after selection.  By default, does nothing."""
        return chosen_children
~~~
<!-- ![Center example image](https://user-images.githubusercontent.com/35067611/62006447-932ffc00-b17b-11e9-8653-f52ff6ea5d93.png "Center"){: .center-image} -->

### Expanding a Tree  
위의 random node expansion 과정을 실제로 몇몇 노드에 적용시켜보자. 가장 핵심적인 알고리즘은 expand_tree_once(self, tree) 함수이다. 이 함수는 새로운 tree를 반환하는 것이 아니라 전달받은 tree 자체에서 mutate을 한다. 이런 in-place mutation 메카니즘은 이 알고리즘을 효율적으로 하는 주요한 요소다.  
~~~python
def expand_tree_once(self, tree):
        """Choose an unexpanded symbol in tree; expand it.  Can be overloaded in subclasses."""
        (symbol, children) = tree
        if children is None:
            # Expand this node
            return self.expand_node(tree)

        # Find all children with possible expansions
        expandable_children = [
            c for c in children if self.any_possible_expansions(c)]

        # `index_map` translates an index in `expandable_children`
        # back into the original index in `children`
        index_map = [i for (i, c) in enumerate(children)
                     if c in expandable_children]

        # Select a random child
        child_to_be_expanded = \
            self.choose_tree_expansion(tree, expandable_children)

        # Expand in place
        children[index_map[child_to_be_expanded]] = \
            self.expand_tree_once(expandable_children[child_to_be_expanded])

        return tree
~~~
### Closing the Expansion  
이제 tree expand는 가능하다. 하지만 언제 어떻게 멈춰야 할까? 최대 사이즈로 derivation tree가 확장된 후 부터는 최소로 트리를 확장시키는 곳에만 적용시킨다. 그러기 위해서 cost를 먼저 계산해 놓는다.  

- symbol_cost() returns the minimum cost of all expansions of a symbol, using expansion_cost() to compute the cost for each expansion.  
- expansion_cost() returns the sum of all expansions in expansions. If a nonterminal is encountered again during traversal, the cost of the expansion is  ∞ , indicating (potentially infinite) recursion.  

### Node Inflation  
fuzzing을 위해 확장을 막 시작할 때는 할 수 있는 최대한의 expansion을 원한다. 즉, expand_node_max_cost()는 minimum cost를 계산해서 tree를 가장 적게 expansion 시키는 함수와는 정반대다. 노드들 중 가장 높은 cost를 가진 노드를 선택하는 함수로써 구현할 수 있다.  

### Putting it all Together  
이로써 tree expansion에도 세 가지 다른 방식 (cost min, cost max, randomly expansion)을 구현했다. 이것을 하나로 합쳐서 fuzzing 과정에서 더 다양한 input을 생성할 수 있도록 유도한다. 이렇게 하면 fuzzing 시간도 훨씬 줄어들고 input들의 size도 훨씬 작아진다. grammar production을 더 효과적으로 컨트롤할 수 있다.  

~~~
tree = self.expand_tree_with_strategy(tree, self.expand_node_max_cost, self.min_nonterminals)
tree = self.expand_tree_with_strategy(tree, self.expand_node_randomly, self.max_nonterminals)
tree = self.expand_tree_with_strategy(tree, self.expand_node_min_cost)
~~~
~~~python
f = GrammarFuzzer(CGI_GRAMMAR, min_nonterminals=3, max_nonterminals=5)
~~~
프로그래머가 정의해주는 min_nonterminal 갯수까지는 max cost로 expand, max_nonterminals 갯수까지는 randomly expand, 그 이후에는 closing expansion을 위해 min cost로 tree를 expand한다.  

### 배운 점  
- derivation tree based grammar fuzzing이 string based보다 효과적이다  
- infinite expansion을 방지할 수 있다  

### 질문  
- display tree function implementation  