---
layout: post
title: "Software Testing Study CH 3-6"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Fuzzing with Generators  

이번 장에서는 문법을 확장시킬 때 함수를 적용하는 것에 대해 공부한다. 이 때 함수란 해당 문법이 실제로 확장 될 때 실행된다. 그 실행 결과로 새로운 input을 생성할 수도 있고 이미 생성된 값들에 대해 validity 체크도 할 수 있다. 또한 생성된 값의 일부를 바꿀 수도 있다. 이렇듯 문법에 함수를 추가하는 것은 다양한 테스팅을 가능토록 해준다.  

### Example: Test a Credit Card System  
먼저 예제를 살펴보자. 흔히 사용하는 신용카드의 카드번호와 결제 금액 정보를 테스팅 값으로 생성시킨다고 해보자. 16자리 카드번호와 결제 금액, 두 가지 정보만 있으면 된다. 이 두 정보는 문법을 통해 쉽게 정의되고 생성될 수 있다.  
~~~python
CHARGE_GRAMMAR = {
    "<start>": ["Charge <amount> to my credit card <credit-card-number>"],
    "<amount>": ["$<float>"],
    "<float>": ["<integer>.<digit><digit>"],
    "<integer>": ["<digit>", "<integer><digit>"],
    "<digit>": crange('0', '9'),

    "<credit-card-number>": ["<digits>"],
    "<digits>": ["<digit-block><digit-block><digit-block><digit-block>"],
    "<digit-block>": ["<digit><digit><digit><digit>"],
}

assert is_valid_grammar(CHARGE_GRAMMAR)
~~~
~~~
['Charge $9.40 to my credit card 7166898575638313',
 'Charge $8.79 to my credit card 6845418694643271',
 'Charge $5.64 to my credit card 6655894657077388',
 'Charge $0.60 to my credit card 2596728464872261',
 'Charge $8.90 to my credit card 2363769342732142']
 ~~~

하지만 실제로 위의 값들로 프로그램 테스팅을 할 때 다음의 두 가지 문제점이 생긴다:  

1. 신용카드의 한도초과 등의 specific한 결제금액을 테스트해볼 수 없다.  
2. 10개 중 9개는 올바른 조합을 가지지 못한 카드번호의 나열로써 사실상 결제가 거부된다. 즉, validity check fail이다.  

위의 문제들을 무시해도 괜찮다. 크고 valid한 수를 생성하는건 시간문제다. 첫 번째 문제점은 문법만 적절히 바꿔주면 해결될 것이다. 하지만 두 번째 문제는 조금 더 깊이 들어간다. 카드번호의 조합은 문법만으로 조합을 만들어 낼 수가 없기 때문이다. 이러한 상황을 해결하기 위해서는 grammar expansion에 함수를 더할 필요가 있다.  

### Attaching Functions to Expansions  
이번 장의 핵심은 문법이 개별적으로 확장될 때 함수를 실행시키는 것이다. 함수가 실행되는 시점은 두 가지일 것이다.  

1. before expansion, replacing the element to be expanded by a computed value.  
2. after expansion, checking generated elements, and possibly also replacing them.  

### Functions Called Before Expansion  
pre option 으로 정의된 함수는 expansion 전에 실행된다. 그 실행 결과 값은 expansion 되기로 했던 값을 대체한다. 예를 들어 위의 신용카드 예제에 pre-expasion function을 다음과 같이 정의하고 문법에 attach할 수 있을 것이다.  
~~~python
def high_charge():
    return random.randint(10000000, 90000000) / 100.0

CHARGE_GRAMMAR.update({
    "<float>": [("<integer>.<digit><digit>", opts(pre=high_charge))],
})
~~~

### Functions Called After Expansion  
post option 으로 정의된 함수는 expansion 후에 실행된다. pre-expansion function과 달리 post-expansion function은 두 가지 역할로 구분하여 실행이 가능하다.  

1. It can serve as a constraint or filter on the expanded values, returning True if the expansion is valid, and False if not; if it returns False, another expansion is attempted.  
2. It can also serve as a repair, returning a string value; like pre-expansion functions, the returned value replaces the expansion.  

즉, 이미 생성된 값에 대해 filter 역할을 하거나, input to be expanded를 repair하여 대체하는 역할을 할 수 있다.  
~~~python
check_credit_card = valid_luhn_checksum
fix_credit_card = fix_luhn_checksum

fix_credit_card("1234567890123456")
~~~
~~~
'1234567890123458'
~~~

### Generating Elements before Expansion  
먼저 pre-expansion function을 구현해야 한다. 기존의 문법에 따라 expansion 될 예정인 값을 함수의 결과값을 통해 대체하는 것이다. 그러기 위해서는 children을 고르는 process_chosen_children() 함수를 hook해야 한다. 그리고 선택된 children에 대해 미리 정의해 놓은 함수를 적용시키면 된다. apply result 함수가 pre-expansion function의 결과를 적용시키는 역할을 한다.  
~~~python
class GeneratorGrammarFuzzer(GeneratorGrammarFuzzer):
    def apply_result(self, result, children):
        if isinstance(result, str):
            children = [(result, [])]
        elif isinstance(result, list):
            symbol_indexes = [i for i, c in enumerate(
                children) if is_nonterminal(c[0])]

            for index, value in enumerate(result):
                if value is not None:
                    child_index = symbol_indexes[index]
                    if not isinstance(value, str):
                        value = repr(value)
                    if self.log:
                        print(
                            "Replacing", all_terminals(
                                children[child_index]), "by", value)

                    # children[child_index] = (value, [])
                    child_symbol, _ = children[child_index]
                    children[child_index] = (child_symbol, [(value, [])])
        elif result is None:
            pass
        elif isinstance(result, bool):
            pass
        else:
            if self.log:
                print("Replacing", "".join(
                    [all_terminals(c) for c in children]), "by", result)

            children = [(repr(result), [])]

        return children
~~~
~~~python
charge_fuzzer = GeneratorGrammarFuzzer(CHARGE_GRAMMAR)
charge_fuzzer.fuzz()
~~~
~~~
'Charge $439383.87 to my credit card 2433506594138520'
~~~
~~~python
amount_fuzzer = GeneratorGrammarFuzzer(
    CHARGE_GRAMMAR, start_symbol="<amount>", log=True)
amount_fuzzer.fuzz()
~~~
~~~
Tree: <amount>
Expanding <amount> randomly
Tree: $<float>
Expanding <float> randomly
<function <lambda> at 0x10806e2f0>() = 382087.72
Replacing <integer>.<digit><digit> by 382087.72
Tree: $382087.72
'$382087.72'
'$382087.72'
~~~

### Support for Python Generators  
파이썬에서는 generator function 개념을 지원한다. generator function은 iterator object를 반환하여 매 반복이 끝날 때 마다 함수의 결과 값을 반환한다. 함수를 끝내는 return 대신에 현재 state를 저장하고 잠시 멈춘다. 다음 반복에서 함수 실행이 성공적으로 이루어지면 resume하여 반복을 계속한다.  
~~~python
def iterate():
    t = 0
    while True:
        t = t + 1
        yield t
~~~
평소에 파이썬에서 range()를 사용하여 반복문을 작성한 것과 동일하게 작동한다.  
~~~python
for i in iterate():
    if i > 10:
        break
    print(i, end=" ")
~~~
~~~
1 2 3 4 5 6 7 8 9 10 
~~~

이번 장에서 다루고 있는 pre-expansion function과 generator function은 무슨 관련이 있으며 어떤 효용이 있는 것일까? pre-expansion function을 반복시켜서 매번 성공적으로 valid하고 서로 다른 input 값을 생성할 수 있도록 할 수 있다. generator를 지원하기 위해서는 process_chosen_children() 함수가 함수가 generator인지 확인해야 한다. 만약 맞다면, run_generator()를 실행시킨다. 만약 run_generator() 함수가 fuzz하는 동안 처음 만난 함수라면 함수를 실행시키고 generator object를 생성, generators attribute에 저장하고 부른다. 물론 문법에 pre-expansion function이 옵션으로 표시되어있어야 한다. 
~~~python
def run_generator(self, expansion, function):
    key = repr((expansion, function))
    if key not in self.generators:
        self.generators[key] = function()
    generator = self.generators[key]
    return next(generator)
~~~

### Checking and Repairing Elements after Expansion  
이제 post-expnasion function의 경우를 이야기해보자. 가장 간단하게 사용하는 방법은 전체 트리가 만들어진 이후에 함수를 실행시켜서 pre function처럼 값들을 대체하도록 하는 것이다. 만약 validity를 체크하는 함수가 False를 반환한다면 새롭게 시작한다.  

### Local Checking and Repairing  
하지만 위와 같이 전체 트리에 대해서 post-expnasion function을 실행하고 틀린 값을 찾아냈을 때 새로운 matching input을 생성시키는 것은 매우 비효율적이다. final subtree가 아니라 partial subtree에 대해서 적용한다면 어떨까? partial subtree가 완성되는 즉시 post-expansion을 부르는 것이다.  

이를 수행하기 위해서 핵심은 local level expansion에서 depth를 0으로 설정해주는 것에 있다. depth가 0이면 그 local subtree는 (잠시동안) 더 이상 expand되지 않을 것이고 post-expnasion function이 역할을 하기 시작한다.  

### Definitions and Uses  
지금까지 다룬 generator와 constraint를 이용해서 훨씬 복잡한 문제들도 해결할 수 있다. 이에 더하여 만약 사전에 정의된 identifier만 사용해서 fuzzing을 하고 싶다면 어떨까?  
previously defined identifiers 정보를 가지고 있는 symbol table를 만든다. 그리고 이 table을 pre/post-expansion function이 매번 실행하기 전에 lookup 하는 방식으로 진행한다.  
### Ordering Expansions  
이제 fuzzing에 사용될 문자를 직접 정의하는 것 까지도 완성됐다. 하지만 여전히 만들어진 definitions에 대해서 order를 컨트롤할 수는 없다. 예를 들어 파이썬에서 ; 문자는 여러 다른 statement를 이어주는 역할을 한다. 그렇다면 ; 문자를 기준으로 앞의 문장이 먼저, 뒤의 문장은 나중에 실행되야 할 것이다. 이런 order 문제도 옵션으로 해결이 가능하다.  
~~~python
CONSTRAINED_VAR_GRAMMAR = extend_grammar(CONSTRAINED_VAR_GRAMMAR, {
    "<statements>": [("<statement>;<statements>", opts(order=[1, 2])),
                     "<statement>"]
})
~~~
~~~python
CONSTRAINED_VAR_GRAMMAR = extend_grammar(CONSTRAINED_VAR_GRAMMAR, {
    "<assignment>": [("<identifier>=<expr>", opts(post=lambda id, expr: define_id(id),
                                                  order=[2, 1]))],
})
~~~
~~~python
def exp_order(expansion):
    """Return the specified expansion ordering, or None if unspecified"""
    return exp_opt(expansion, 'order')
~~~

### 배운 점  
- pre/post-expansion function을 문법에 적용하면 더 좋은 테스트케이스를 만들어 낼 수 있다.  
- generator function을 사용하여 pre/post-expansion function의 효과를 향상시킬 수 있다.  