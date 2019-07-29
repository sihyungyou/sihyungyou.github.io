---
layout: post
title: "Software Testing Study CH 3-1"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Fuzzing with Grammars  

CH 2-3에서 test generatio의 속도를 올리고 최대한 valid input 범위 내에서 fuzzing을 할 수있도록 하는 Mutation based fuzzing에 대해 공부했다. 여기서 한 단계 더 발전시켜 프로그램에 specification of legal input을 만들어주려 한다. Fuzzing with grammars를 통해서 하는데 이는 valid input을 생성시킬 뿐 아니라 더 복잡한 입력일수록 효과를 발휘하게 된다.  

프로그램에 대한 입력은 온갖 (실패를 포함해서) program behavior를 야기한다. 그렇기 때문에 입력들을 효과적, 체계적으로 관리하는 것은 매우 중요하다. Grammars는 syntatical structure of input을 설명하는데 매우 좋으며 재귀적으로 그 표현을 간단하게 정리할 수 있다는 장점이 있다.  

### Rules and Expansions  
Grammar는 start symbol로 시작해서 set of expansion으로 그 규칙을 확장시켜 나간다. 예를 들어, 아래는 두 자리 숫자를 나타내는 grammar이다.
~~~
<start> ::= <digit><digit>
<digit> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
~~~

여기서 ::= 표시는 "lhs가 rhs로 대체될 수 있다"를 의미한다. grammar가 재귀적으로 불릴 수 있다는 것은 아래와 같은 경우를 의미한다.  
~~~
<start>   ::= <number>
<number>  ::= <integer> | +<integer> | -<integer>
<integer> ::= <digit> | <digit><integer>
<digit>   ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
~~~

위의 예들과 같은 grammar에서 start로 시작하여 다른 symbol들로 대안들을 찾아나가는 과정을 반복한다면 빠르게 valid expression을 생성시킬 수 있을 것이다. 이 과정이 곧 grammar fuzzing이고 이는 복잡하지만 문법에서 허용되는 입력을 빠르게 생성시키는데 매우 효과적이다. 다만, context-free characterization을 위해서 lhs에는 항상 single symbol이 온다는 것에 유념하자. 또한 앞으로 grammar fuzzer를 구현하면서 start symbol로 시작해 expandsion읋 해 나갈 텐데 무한한 입력의 경우를 배제하기 위해서 max_nonterminals(i.e., anything between < and >, except spaces)를 정해두고 한다.  
~~~python
def simple_grammar_fuzzer(grammar, start_symbol=START_SYMBOL,
                          max_nonterminals=10, max_expansion_trials=100,
                          log=False):
    term = start_symbol
    expansion_trials = 0

    while len(nonterminals(term)) > 0:
        symbol_to_expand = random.choice(nonterminals(term))
        expansions = grammar[symbol_to_expand]
        expansion = random.choice(expansions)
        new_term = term.replace(symbol_to_expand, expansion, 1)

        if len(nonterminals(new_term)) < max_nonterminals:
            term = new_term
            if log:
                print("%-40s" % (symbol_to_expand + " -> " + expansion), term)
            expansion_trials = 0
        else:
            expansion_trials += 1
            if expansion_trials >= max_expansion_trials:
                raise ExpansionError("Cannot expand " + repr(term))

    return term
~~~ 

### Grammars as Mutation Seeds  
Grammar fuzzing의 가장 유용한 점은 거의 모든 생성되는 입력이 valid 하다는 점이다. syntatical 관점에서 보면 사실 "항상" valid하다. 단, grammar로도 표현할 수 없는 semantical한 문제는 남아있다. 예를 들면 url grammar를 구현한다고 했을 때 포트넘버는 1024~2048사이여야 하는데 이런 것은 grammar로 쓰기 쉽지 않다. 이런 문제를 해결하기 위해 grammar에 여러 제약을 걸 수 있는데 이는 나중에 다루도록 하겠다. 지금은 다른 방법으로 이 문제를 해결하려 한다. 바로 mutation-based fuzzing과 grammar fuzzing의 강점을 합치는 것이다. 이로써 valid 할 뿐 아니라 valid-invalid input의 영역을 체크할 수 있다.  

### Extending Grammars  
먼저 입력 문자열에 반드시 < 혹은 > 문자를 포함해야 하는 경우를 고려해야한다. 현재 구현으로는 non terminal symbol들은 <, >로 구분되고 있기 때문이다. left angle, right angle를 정의해줌으로써 쉽게 해결할 수 있다.  
~~~python
simple_nonterminal_grammar = {
    "<start>": ["<nonterminal>"],
    "<nonterminal>": ["<left-angle><identifier><right-angle>"],
    "<left-angle>": ["<"],
    "<right-angle>": [">"],
    "<identifier>": ["id"]  # for now
}
~~~

새로운 문법을 만들기 위해서 (extending) 먼저 기존의 것을 복사한 후 새로운 문법을 추가하는 방식으로 진행한다.  
~~~python
nonterminal_grammar = copy.deepcopy(simple_nonterminal_grammar)
nonterminal_grammar["<identifier>"] = ["<idchar>", "<identifier><idchar>"]
nonterminal_grammar["<idchar>"] = ['a', 'b', 'c', 'd']  # for now
~~~

### Simplify the expressions  
1. Character classes  
Grammar 작성 시, idchar를 한 문자 한 문자 다 써주기에는 표현이 너무 길어지고 너무 힘든 작업이다. 아래처럼하면 조금은 스마트하게 표현을 바꿀 수 있다.  
~~~python
nonterminal_grammar = extend_grammar(nonterminal_grammar, {"<idchar>": srange(string.ascii_letters) + srange(string.digits) + srange("-_") })
~~~

2. Grammar Shortcuts  
symbol들의 재귀를 표현하기 위해서 지금까지는 아래와 같이 문법을 작성했다.  
~~~
['<idchar>', '<identifier><idchar>']
~~~
하지만 shortcut을 사용하면 더 간결하게 표현할 수 있다.  
~~~
The form <symbol>? indicates that <symbol> is optional – that is, it can occur 0 or 1 times.
The form <symbol>+ indicates that <symbol> can occur 1 or more times repeatedly.
The form <symbol>* indicates that <symbol> can occur 0 or more times. (In other words, it is an optional repetition.)
To make matters even more interesting, we would like to use parentheses with the above shortcuts. Thus, (<foo><bar>)? indicates that the sequence of <foo> and <bar> is optional.
~~~

### Checking Grammars  
grammar가 string representation이기 때문에 에러가 있는 경우가 많을 수 밖에 없다. 이러한 케이스를 잡아내기 위해서 여러가지 valid grammar checker (helper함수)를 만들어 놓는다. 코드가 길어 복잡해 보이지만 단순한 에러 체크 로직의 반복이다.  
~~~python
def is_valid_grammar(grammar, start_symbol=START_SYMBOL, supported_opts=None):
    defined_nonterminals, used_nonterminals = \
        def_used_nonterminals(grammar, start_symbol)
    if defined_nonterminals is None or used_nonterminals is None:
        return False

    # Do not complain about '<start>' being not used,
    # even if start_symbol is different
    if START_SYMBOL in grammar:
        used_nonterminals.add(START_SYMBOL)

    for unused_nonterminal in defined_nonterminals - used_nonterminals:
        print(repr(unused_nonterminal) + ": defined, but not used",
              file=sys.stderr)
    for undefined_nonterminal in used_nonterminals - defined_nonterminals:
        print(repr(undefined_nonterminal) + ": used, but not defined",
              file=sys.stderr)

    # Symbols must be reachable either from <start> or given start symbol
    unreachable = unreachable_nonterminals(grammar, start_symbol)
    msg_start_symbol = start_symbol
    if START_SYMBOL in grammar:
        unreachable = unreachable - \
            reachable_nonterminals(grammar, START_SYMBOL)
        if start_symbol != START_SYMBOL:
            msg_start_symbol += " or " + START_SYMBOL
    for unreachable_nonterminal in unreachable:
        print(repr(unreachable_nonterminal) + ": unreachable from " + msg_start_symbol,
              file=sys.stderr)

    used_but_not_supported_opts = set()
    if supported_opts is not None:
        used_but_not_supported_opts = opts_used(
            grammar).difference(supported_opts)
        for opt in used_but_not_supported_opts:
            print(
                "warning: option " +
                repr(opt) +
                " is not supported",
                file=sys.stderr)

    return used_nonterminals == defined_nonterminals and len(unreachable) == 0
~~~

### 배운 점  
- syntactically valid inputs을 생성하는 데에 grammar는 매우 효율적이고 강력한 도구다.  
- grammar fuzzing을 통해 생성된 입력 후보들을 seeds로 두고 mutation-based fuzzing과 합치면 테스팅의 효과를 높일 수 있다.  