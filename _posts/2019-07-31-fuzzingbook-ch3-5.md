---
layout: post
title: "Software Testing Study CH 3-5"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Probabilistic Grammar Fuzzing  

이제 문법에 개연성(probabilities)을 더함으로써 더욱 강력하게 만들어 보자. probabilities는 fuzzing 시, 각 요소가 얼마나 많이 생성되어야 하는지 알려줄 수 있다. 결과적으로 특정한 기능에 대한 테스팅을 가능케해준다. 이 장에서는 probabilities를 어떻게 적용하는지, 어떻게 샘플 데이터로부터 얻어낼 수 있는지 공부한다.  

### The Law of Leading Digits  
먼저 probabilities의 필요성에 대해서 이야기해보자. 지금까지의 예시들을 보면 프로그램에 의해 생성된 입력값들이 자연적(natural)으로 생성되는, 혹은 일반적으로 실생활에서 쓰이는 값들과는 조금 다르다는 것을 알 수 있다. 이는 실생활에서 쓰이는 numerical data에는 특정 값이 자주 나오거나 별로 등장하지 않는 경우가 발생하기 때문이다. 실제로 평균적으로 leading digit으로써 1이 8이나 9보다 6배나 자주 쓰인다. (Benford's law) 실생활에서 쓰이는 데이터라 함은 거리 주소, 주식 가격, 부동산 가격, 인구, 사망률 등을 의미한다. 즉, 과학적 논문에 거짓된 데이터를 랜덤한 수를 생성해서 사용하게 되면 높은 확률로 Benford's law를 위배해서 적발된다는 말이다. 그렇다면 Benford's law에 고착하는 수를 우리가 만들고 싶다면 어떻게 해야 할까? 이번 장에서 다룰 probabilities를 grammar에 적용시키는 방법을 사용하면 된다.  

leading digit을 d라고 할 때, probability P(d)는 다음의 공식을 통해 계산된다 :  
P(d)=log10(d+1)−log10(d)  

### Specifying Probabilities  
이번 장의 목표는 문법의 개별적 expansion에 probabilities를 부여해서 특정 expansion이 다른 것들 보다 더 favored 되도록 만드는 것이다. 이것은 natural-looking numbers를 찾는 데에 유용할 뿐 아니라 특정한 목표를 갖고 테스팅의 방향성을 조정할 때도 도움이 된다. 만약 프로그램에서 약간의 코드 수정이 있었다면 프로그래머는 아마 그 부분에 대해서 더 정밀한 테스팅을 원할 것이다. 이런 경우에 바뀐 코드와 관련있는 입력값들에 대한 probabilities가 더 테스팅을 효과적으로 해줄 것이다. probabilities를 표기하는 방법은 이전과 마찬가지로 annotation을 표기하는 방식으로 진행해보자. 아래는 EXPR_GRAMMAR의 일부 expansion에 probabilities를 부여한 것이다.  
~~~python
PROBABILISTIC_EXPR_GRAMMAR = {
    # Benford's law: frequency distribution of leading digits
    "<leaddigit>":
        [("1", opts(prob=0.301)),
         ("2", opts(prob=0.176)),
         ("3", opts(prob=0.125)),
         ("4", opts(prob=0.097)),
         ("5", opts(prob=0.079)),
         ("6", opts(prob=0.067)),
         ("7", opts(prob=0.058)),
         ("8", opts(prob=0.051)),
         ("9", opts(prob=0.046)),
         ],
}
~~~

### Distributing Probabilities  
~~~python
def prob_distribution(probabilities, nonterminal="<symbol>"):
    epsilon = 0.00001

    number_of_unspecified_probabilities = probabilities.count(None)
    if number_of_unspecified_probabilities == 0:
        assert abs(sum(probabilities) - 1.0) < epsilon, \
            nonterminal + ": sum of probabilities must be 1.0"
        return probabilities

    sum_of_specified_probabilities = 0.0
    for p in probabilities:
        if p is not None:
            sum_of_specified_probabilities += p
    assert 0 <= sum_of_specified_probabilities <= 1.0, \
        nonterminal + ": sum of specified probabilities must be between 0.0 and 1.0"

    default_probability = ((1.0 - sum_of_specified_probabilities)
                           / number_of_unspecified_probabilities)
    all_probabilities = []
    for p in probabilities:
        if p is None:
            p = default_probability
        all_probabilities.append(p)

    assert abs(sum(all_probabilities) - 1.0) < epsilon
    return all_probabilities
~~~

### Checking Probabilities  
probabilities가 적용된 문법에 대해서 validity, consistency check도 되어야 한다.  
~~~python
def is_valid_probabilistic_grammar(grammar, start_symbol=START_SYMBOL):
    if not is_valid_grammar(grammar, start_symbol):
        return False

    for nonterminal in grammar:
        expansions = grammar[nonterminal]
        prob_dist = exp_probabilities(expansions, nonterminal)

    return True
~~~
~~~python
with ExpectError():
    assert is_valid_probabilistic_grammar({"<start>": [("1", opts(prob=0.5))]})
~~~
~~~
AssertionError: <start>: sum of probabilities must be 1.0 (expected)
~~~

### Expanding by Probability  
~~~python
class ProbabilisticGrammarFuzzer(ProbabilisticGrammarFuzzer):
    def choose_node_expansion(self, node, possible_children):
        (symbol, tree) = node
        expansions = self.grammar[symbol]
        probabilities = exp_probabilities(expansions)

        weights = []
        for child in possible_children:
            expansion = all_terminals((node, child))
            child_weight = probabilities[expansion]
            if self.log:
                print(repr(expansion), "p =", child_weight)
            weights.append(child_weight)

        if sum(weights) == 0:
            # No alternative (probably expanding at minimum cost)
            weights = None

        return random.choices(
            range(len(possible_children)), weights=weights)[0]
~~~
현재 위 코드는 probabilistic grammar fuzzer 이지만 probability annotation을 표시하는 것 외에는 non-probabilistic grammar fuzzer 처럼 작동한다. 실제로 natural number를 생성시켜 보면 Benford's law에 따라 수가 분포될 것이다.  

### Directed Fuzzing  
지금까지 어떻게 문법에 probabilities를 특정하는지 봤다. 이제 expansion을 할 때 child를 고르는 방법에서 probabilities를 기준으로 가중치를 두고 고른다.  
~~~python
probabilistic_url_grammar = extend_grammar(URL_GRAMMAR)
set_prob(probabilistic_url_grammar, "<scheme>", "ftps", 0.8)
assert is_valid_probabilistic_grammar(probabilistic_url_grammar)
~~~
~~~python
prob_url_fuzzer = ProbabilisticGrammarFuzzer(probabilistic_url_grammar)
for i in range(10):
    print(prob_url_fuzzer.fuzz())
~~~
~~~
ftps://cispa.saarland:80
ftps://user:password@cispa.saarland/
ftps://fuzzingbook.com/abc
ftps://fuzzingbook.com/x48
ftps://user:password@fuzzingbook.com/
ftps://www.google.com:2?x18=8
ftps://user:password@www.google.com:6
ftps://user:password@cispa.saarland/def
ftps://user:password@cispa.saarland/def?def=52
ftps://user:password@cispa.saarland/
~~~

참고로 probability가 0인 expansion들도 여전히 프로그램이 돌아가면 expand 된다. 이런 일은 closing grammar fuzzer 단계에서 일어날 수 있다. probability가 0라도 closing expansion에 필수적이라면 프로그램이 take 하는 것이다.  

### Probabilities in Context  
specified probabilities가 우리에게 어떤 expansion을 얼마나 자주 할 지 control 측면에서 큰 도움이 되지만 충분치는 못하다.  
~~~python
IP_ADDRESS_GRAMMAR = {
    "<start>": ["<address>"],
    "<address>": ["<octet>.<octet>.<octet>.<octet>"],
    # ["0", "1", "2", ..., "255"]
    "<octet>": decrange(0, 256)
}
~~~
~~~python
probabilistic_ip_address_grammar = extend_grammar(IP_ADDRESS_GRAMMAR)
set_prob(probabilistic_ip_address_grammar, "<octet>", "127", 0.8)
~~~
~~~
'127.127.127.127'
~~~
네 개의 octet에 각각 다른 probabilities을 주려면 어떻게 해야 할까? 이전 장에서 다뤘던 맥락(context) 개념을 생각해보면 각 요소를 duplicate 하는 것이다.  
~~~python
set_prob(probabilistic_ip_address_grammar, "<octet-1>", "127", 1.0)
set_prob(probabilistic_ip_address_grammar, "<octet-2>", "0", 1.0)
~~~

### Learning Probabilities from Samples  
Probabilities가 매번 메뉴얼하게 정해질 필요는 없다. 다른 소스로부터 학습할 수 도 있기 때문이다. 예를 들면 주어진 샘플 데이터에서 자주 나오는 것, 거의 나오지 않는 것, 혹은 일부를 발췌하여 probabilities를 정할 수도 있다.  

1. Test common features. The idea is that during testing, one may want to focus on frequently occurring (or frequently used) features first, to ensure correct functionality for the most common usages.  
2. Test uncommon features. Here, the idea is to have test generation focus on features that are rarely seen (or not seen at all) in inputs. This is the same motivation as with grammar coverage, but from a probabilistic standpoint.  
3. Focus on specific slices. One may have a set of inputs that is of particular interest (for instance, because they exercise a critical functionality, or recently have discovered bugs). Using this learned distribution for fuzzing allows us to focus on precisely these functionalities of interest.  

### Counting Expansions  
그렇다면 주어진 입력값들 중에 어던 expansion이 자주 나오고, 거의 나오지 않는지 count 해야 할 것이다. 이를 위해서는 먼저 string input을 derivation tree로 변환해야 한다.  

![Center example image](https://user-images.githubusercontent.com/35067611/62181010-d1503a00-b38c-11e9-988b-7ee4ee5dcaff.png "Center"){: .center-image}  

위와 같이 tree 형태로 변환 하고 나서는 각각의 expansion을 셀 수 있다. 0은 2개, 1은 한개, 127도 한개이므로 0이 50%, 1과 127은 각각 25%의 portion을 차지한다. 나머지는 0%이다. 샘플 데이터로부터 얻은 이 수치가 우리가 grammar에 적용하고 싶은 probabilities가 되는 것이다.  

### Exploration vs. Exploitation  

By learning (and re-learning) probabilities from a subset of sample inputs, we can specialize fuzzers towards the properties of that subset – in our case, inputs that contain percentage signs and valid hexadecimal letters. The degree to which we can specialize things is induced by the number of variables we can control – in our case, the probabilities for the individual rules. Adding more context to the grammar, as discussed above, will increase the number of variables, and thus the amount of specialization.

A high degree of specialization, however, limits our possibilities to explore combinations that fall outside of the selected scope, and limit our possibilities to find bugs induced by these combinations. This tradeoff is known as exploration vs. exploitation in machine learning – shall one try to explore as many (possibly shallow) combinations as possible, or focus (exploit) specific areas? In the end, it all depends on where the bugs are, and where we are most likely to find them. Assigning and learning probabilities allows us to control the search strategies – from the common to the uncommon to specific subsets.  

### Lessons Learned  
By specifying probabilities, one can steer fuzzing towards input features of interest.
Learning probabilities from samples allows one to focus on features that are common or uncommon in input samples.
Learning probabilities from a subset of samples allows one to produce more similar inputs.