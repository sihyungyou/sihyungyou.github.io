---
layout: post
title: "Software Testing Study CH 3-6"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Fuzzing with Generators  

In this chapter, we show how to extend grammars with functions – pieces of code that get executed during grammar expansion, and that can generate, check, or change elements produced. Adding functions to a grammar allows for very versatile test generation, bringing together the best of grammar generation and programming.

### Example: Test a Credit Card System  
Suppose you work with a shopping system that – among several other features – allows customers to pay with a credit card. Your task is to test the payment functionality.

To make things simple, we will assume that we need only two pieces of data – a 16-digit credit card number and an amount to be charged. Both pieces can be easily generated with grammars, as in the following:
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

However, when actually testing our system with this data, we find two problems:

1. We'd like to test specific amounts being charged – for instance, amounts that would excess the credit card limit.  
2. We find that 9 out of 10 credit card numbers are rejected because of having an incorrect checksum. This is fine if we want to test rejection of credit card numbers – but if we want to test the actual functionality of processing a charge, we need valid numbers.  

We could go and ignore these issues; after all, eventually, it is only a matter of time until large amounts and valid numbers are generated. As it comes to the first concern, we could also address it by changing the grammar appropriately – say, to only produce charges that have at least six leading digits. However, generalizing this to arbitrary ranges of values will be cumbersome.

The second concern, the checksums of credit card numbers, however, runs deeper – at least as far as grammars are concerned, is that a complex arithmetic operation like a checksum cannot be expressed in a grammar alone – at least not in the context-free grammars we use here. (In principle, one could do this in a context–sensitive grammar, but specifying this would be no fun at all.) What we want is a mechanism that allows us to attach programmatic computations to our grammars, bringing together the best of both worlds.

### Attaching Functions to Expansions  
The key idea of this chapter is to extend grammars such that one can attach Python functions to individual expansions. These functions can be executed

1. before expansion, replacing the element to be expanded by a computed value  
2. after expansion, checking generated elements, and possibly also replacing them.  

### Functions Called Before Expansion  
A function defined using the pre option is invoked before expansion of  s  into  e . Its value replaces the expansion  e  to be produced. To generate a value for the credit card example, above, we could define a pre-expansion generator function
~~~python
def high_charge():
    return random.randint(10000000, 90000000) / 100.0

CHARGE_GRAMMAR.update({
    "<float>": [("<integer>.<digit><digit>", opts(pre=high_charge))],
})
~~~

### Functions Called After Expansion  
A function defined using the post option is invoked after expansion of  s  into  e , passing the expanded values of the symbols in  e  as arguments. A post-expansion function can serve in two ways:

1. It can serve as a constraint or filter on the expanded values, returning True if the expansion is valid, and False if not; if it returns False, another expansion is attempted.  
2. It can also serve as a repair, returning a string value; like pre-expansion functions, the returned value replaces the expansion.  
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
