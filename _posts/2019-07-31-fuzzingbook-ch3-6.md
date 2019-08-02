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

Our first task will be implementing the pre-expansion functions – that is, the function that would be invoked before expansion to replace the value to be expanded. To this end, we hook into the process_chosen_children() method, which gets the selected children before expansion. We set it up such that it invokes the given pre function and applies its result on the children, possibly replacing them.  
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
The Python language has its own concept of generator functions, which we of course want to support as well. A generator function in Python is a function that returns a so-called iterator object which we can iterate over, one value at a time.

To create a generator function in Python, one defines a normal function, using the yield statement instead of a return statement. While a return statement terminates the function, a yield statement pauses its execution, saving all of its state, to be resumed later for the next successive calls.
~~~python
def iterate():
    t = 0
    while True:
        t = t + 1
        yield t
~~~

We can use iterate in a loop, just like the range() function (which also is a generator function):
~~~python
for i in iterate():
    if i > 10:
        break
    print(i, end=" ")
~~~
~~~
1 2 3 4 5 6 7 8 9 10 
~~~

We can also use iterate() as a pre-expansion generator function, ensuring it will create one successive integer after another:

To support generators, our process_chosen_children() method, above, checks whether a function is a generator; if so, it invokes the run_generator() method. When run_generator() sees the function for the first time during a fuzz_tree() (or fuzz()) call, it invokes the function to create a generator object; this is saved in the generators attribute, and then called. Subsequent calls directly go to the generator, preserving state.

### Checking and Repairing Elements after Expansion  
Let us now turn to our second set of functions to be supported – namely, post-expansion functions. The simplest way of using them is to run them once the entire tree is generated, taking care of replacements as with pre functions. If one of them returns False, however, we start anew.  
