---
layout: post
title: "Software Testing Study CH 3-7"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Greybox Fuzzing with Grammars  

이번 장에서는 syntactic fuzzing 기술에서의 두 가지 중요한 확장을 소개한다.  
parsing과 fuzzing with grammars를 합치는 것을 먼저 보게 된다. 둘을 합침으로써 syntactical correctness를 보존하고, 이미 존재하는 input's fragments를 재사용 하면서 mutate 할 수 있게 된다. 실제로 JavaScript interpreter의 2600개 이상의 버그를 찾은 Langfuzz fuzzer가 이러한 방법으로 만들어졌다.  

이전 장 까지는 black-box 방식으로 문법을 사용해왔다. 즉, 프로그램이 어떻게 테스팅 되는가 와는 무관하게 문법을 이용해 input generation을 했던 것이다. 이번 장에서는 mutational greybox fuzzing with grammars를 소개한다. 이 기술은 프로그램 테스팅의 피드백을 활용하여 특정한 목표로 나아가는 테스팅 생성을 가능케 한다. lexical greybox fuzzing에서 보았던 것 처럼 지금 말하는 피드백이란 coverage를 의미하고, 이는 지금까지 그랬듯, 아직 가보지 못한 코드부분까지 우리를 guide하되, grammar-based feature를 여전히 가지고 갈 것이다.  

### Background  
먼저 mutational fuzzer의 몇 가지 기본적인 성분들을 복습해보자  

- Seed. A seed is an input that is used by the fuzzer to generate new inputs by applying a sequence of mutations.  
- Mutator. A mutator implements a set of mutation operators that applied to an input produce a slightly modified input.  
- PowerSchedule. A power schedule assigns energy to a seed. A seed with higher energy is fuzzed more often throughout the fuzzing campaign.  
- MutationFuzzer. Our mutational blackbox fuzzer generates inputs by mutating seeds in an initial population of inputs.  
- GreyboxFuzzer. Our greybox fuzzer dynamically adds inputs to the population of seeds that increased coverage.  
- FunctionCoverageRunner. Our function coverage runner collects coverage information for the execution of a given Python function.  

### Building a Keyword Dictionary  
HTML parser을 fuzz하기 위해서는 mutational fuzzer에 중요한 html의 문법상 태그들을 넣어주는 것이 유용하다. 
~~~python
dict_mutator = DictMutator(["<a>","</a>","<a/>", "='a'"])
~~~

![Center example image](https://user-images.githubusercontent.com/35067611/62351047-70b52e80-b53f-11e9-9cc3-c32ac8a15e3c.png "Center"){: .center-image}  

이렇게 fuzzer에게 중요한 키워드를 알려주는 것은 많은 coverage를 빠르게 성취하는데 큰 도움이 된다.  

### Fuzzing with Input Fragments  
dictionaries 자료구조가 주요 키워드를 fuzzer에 심는 데에 유용하지만 이것이 생성되는 입력들의 구조적(structural) integrity를 보장하지는 못한다. 대신 문법을 이용해서, 다음의 세 단계를 통해 이러한 input structure을 fuzzer에게 알려주어야 한다.

1. parses the seed inputs,  
2. disassembles them into input fragments, and  
3. generates new files by reassembling these fragments according to the rules of the grammar.

이렇게 만들어지는 parsing과 fuzzing의 결합은 매우 강력하다.  

### Parsing and Recombining HTML  
HTML input 예제로 계속 이어가 보자. 유효한 HTML input을 만들기 위해서는 먼저 문법을 정의해줘야 한다. context-free grammar는 열고 닫는 tag를 필요로 하지 않지만 이런 context-sensitive feature가 input fragment에서 유지되는 것을 볼 수 있다.  

~~~python
XML_TOKENS = {"<id>","<text>"}

XML_GRAMMAR = {
    "<start>": ["<xml-tree>"],
    "<xml-tree>": ["<text>",
                   "<xml-open-tag><xml-tree><xml-close-tag>", 
                   "<xml-openclose-tag>", 
                   "<xml-tree><xml-tree>"],
    "<xml-open-tag>":      ["<<id>>", "<<id> <xml-attribute>>"],
    "<xml-openclose-tag>": ["<<id>/>", "<<id> <xml-attribute>/>"],
    "<xml-close-tag>":     ["</<id>>"],
    "<xml-attribute>" :    ["<id>=<id>", "<xml-attribute> <xml-attribute>"],
    "<id>":                ["<letter>", "<id><letter>"],
    "<text>" :             ["<text><letter_space>","<letter_space>"],
    "<letter>":            srange(string.ascii_letters + string.digits +"\""+"'"+"."),
    "<letter_space>":      srange(string.ascii_letters + string.digits +"\""+"'"+" "+"\t"),
}
~~~

### Building the Fragment Pool  
이제 input-structure-aware mutator을 만들 준비가 끝났다. 먼저 mutator를 비어있는 fragment pool로 초기화 시켜준다. 역시 dictionary 자료구조를 이용하기 때문에 문법에서 정의된 key와 symbol의 쌍으로 이루어진다.  

FragmentMutator는 fragment를 재귀적으로 추가한다. fragment는 parse tree의 subtree이며 현재 노드와 자식 노드의 symbol로 구성된다. token, terminals, 혹은 문법에 정의되지 않은 symbol 들로 시작하는 fragment들은 배제할 수 있다.  

add_to_fragment_pool() 함수는 이름 그대로 모든 fragments를 pool에 추가한다. seed parsing이 성공적으로 끝난다면 seed.has_structure가 True로 세팅된다. parsing에 실패하면 False이다.  

~~~python
class FragmentMutator(FragmentMutator):
    def add_fragment(self, fragment):
        """Recursively adds fragments to the fragment pool"""
        (symbol, children) = fragment
        if not self.is_excluded(symbol):
            self.fragments[symbol].append(fragment)
            for subfragment in children:
                self.add_fragment(subfragment)

    def add_to_fragment_pool(self, seed):
        """Adds all fragments of a seed to the fragment pool"""
        try: # only allow quick parsing of 200ms max
            signal.setitimer(signal.ITIMER_REAL, 0.2)
            seed.structure = next(self.parser.parse(seed.data))
            signal.setitimer(signal.ITIMER_REAL, 0)

            self.add_fragment(seed.structure)
            seed.has_structure = True
        except (SyntaxError, Timeout):
            seed.has_structure = False
            signal.setitimer(signal.ITIMER_REAL, 0)
~~~

fragment pool for simple HTML seed input result : 
~~~
<start>
|-<html><header><title>Hello</title></header><body>World<br/></body></html>
<xml-tree>
|-<html><header><title>Hello</title></header><body>World<br/></body></html>
|-<header><title>Hello</title></header><body>World<br/></body>
|-<header><title>Hello</title></header>
|-<title>Hello</title>
|-Hello
|-<body>World<br/></body>
|-World<br/>
|-World
|-<br/>
<xml-open-tag>
|-<html>
|-<header>
|-<title>
|-<body>
<xml-openclose-tag>
|-<br/>
<xml-close-tag>
|-</title>
|-</header>
|-</body>
|-</html>
<xml-attribute>
<id>
<text>
<letter>
<letter_space>
~~~

위의 결과를 보면 문법 속 symbol들로 부터 많은 fragment를 얻었다는 걸 알 수 있다.  
문법의 각 symbol들을 FragmentMutator가 fragment로 세팅하고, 이들은 입력값을 mutate하기 위해 첫번째로 하는 parsing에 얻어진다.  

### Fragment-Based Mutation  
fragment pool에 있는 여러 fragments들을 새로운 input을 생성하는데 사용할 수 있다. 모든 seed는 fragment로 쪼개어지고, 저장된다(memoized)  

swap_fragments() 함수는 첫번째 structural mutation 함수로써 주어진 seed에서 랜덤한 fragment를 골라서 pool에 있는 것과 랜덤하게 바꾼다. 물론 두 fragment의 start symbol이 같아야 한다. 예를 들어 닫는 tag를 seed에서 골랐다면 pool에서도 닫는 tag를 골라야 한다. random fragment를 고르기 위해서 먼저 mutator는 총 몇 개의 fragment가 있는지 센다.  

2 ~ total number of fragment 사이의 수 중 랜덤한 수를 structural mutator가 고른다. 그리고 재귀적으로 새로운 fragment를 만들어낸다. 이들은 새로운 seed로 반환된다. 이런 논리는 random fragment를 삭제할 때도 이용된다.  

~~~python
class FragmentMutator(FragmentMutator):
    def recursive_swap(self, fragment):
        """Recursively finds the fragment to swap."""
        symbol, children = fragment
        if self.is_excluded(symbol):
            return symbol, children

        self.to_swap -= 1
        if self.to_swap == 0: 
            return random.choice(list(self.fragments[symbol]))
        return symbol, list(map(self.recursive_swap, children))

    def recursive_delete(self, fragment):
        """Recursively finds the fragment to delete"""
        symbol, children = fragment
        if self.is_excluded(symbol):
            return symbol, children

        self.to_delete -= 1
        if self.to_delete == 0: 
            return symbol, []
        return symbol, list(map(self.recursive_delete, children))

    def swap_fragment(self, seed):
        """Substitutes a random fragment with another with the same symbol"""
        if seed.has_structure:
            n_nodes = self.count_nodes(seed.structure)
            self.to_swap = random.randint(2, n_nodes)
            new_structure = self.recursive_swap(seed.structure)

            new_seed = Seed(tree_to_string(new_structure))
            new_seed.has_structure = True
            new_seed.structure = new_structure
            return new_seed
        return seed
~~~

이제 structure-aware fuzzing의 모든 준비물은 마련됐다. 현재 mutator는 모든 seed를 fragment로 쪼개고 pool에 넣는다. 그리고 같은 type을 가진 fragment끼리 swap하여 새로운 seed를 만들고, 주어졌던 seed에서는 랜덤한 fragment를 삭제한다. 이 과정은 높은 확률의 validity를 보장한다.  

### Integration with Greybox Fuzzing  
fragment-level blackbox fuzzer (LangFuzzer)는 더 많은 valid input을 만들어 내지만 code coverage 측면에서는 byte-level fuzzer보다 효율이 떨어진다. 그래서 생성된 입력들 중 주어진 문법을 고수하지 않는 경우도 있다.  

fragment-level balckbox fuzzing과 byte-level greybox fuzzing의 장점들을 취하기 위해 이 둘을 합치려 한다. 추가적인 coverage 피드백은 더 많은 coverage를 빠르게 얻을 수 있도록 도와줄 것이다. greybox fuzzer는 seed population에 code coverage를 증가시킬 모든 생성된 입력값을 추가한다. 

~~~python
class GreyboxGrammarFuzzer(GreyboxFuzzer):
    def create_candidate(self):
        """Returns an input generated by structural mutation of a seed in the population"""
        seed = self.schedule.choose(self.population)

        # Structural mutation
        trials = random.randint(0,4)
        for i in range(trials):
            seed = self.tree_mutator.mutate(seed)

        # Byte-level mutation
        candidate = seed.data
        if trials == 0 or not seed.has_structure or 1 == random.randint(0, 1):
            dumb_trials = min(len(seed.data), 1 << random.randint(1,5))
            for i in range(dumb_trials):
                candidate = self.mutator.mutate(candidate)
        return candidate
~~~

structural greybox fuzzer은
1. fragment-based LangFuzzer보다 빠르다.  
2. fragment-based LangFuzzer, vanilla blackbox mutational fuzzer 보다 더 많은 code coverage를 얻을 수 있다.  
3. vanilla blackbox mutational fuzzer 보다 더 적은 valid input을 만든다.  

### Mutating Invalid Seeds  
지금까지 생성한 input들은 대부분 fuzzer에게 주어진 문법에 위배되는 경우가 더 많았다. fragment-based mutator을 적용하기 위해서는 seed를 성공적으로 parse해야 한다. 그렇지 않으면 지금까지 만든 fragment-based approach가 무의미하기 때문이다. 그렇다면 어떻게 올바르게 parse될 수 없는 seed로부터 structure를 파생시킬 수 있을까?  

이를 위해 region-based mutation이라는 아이디어를 가져온다. 먼저 AFLSmart structural greybox fuzzer를 통해 탐험(explore)한다. 이 fuzzer는 byte-level, fragment-based, region-based mutation, validity-based power schedule로 구현되어 있다. region은 grammar의 symbol과 매치되는 input의 byte의 연속적인 시퀀스를 의미한다.  

### Determining Symbol Regions  
방법은 이렇다 : 먼저 parse table을 만든다. 문자열의 각 문자는 potential symbol과 region of neighboring letter 정보를 줄 것이다. input fragment와는 다르게 input regions는 parser가 실패하더라도 전체 parse 트리를 만들 수 있다.  

### Region-based Mutation  
invalid seed를 fuzz 하기 위해서는 region-based mutator가 seed에 있는 region의 문법들 속 symbol들과 연결된다. 앞에 나왔던 add_to_fragment_pool() 함수는 먼저 seed로부터 fragment 정보를 캔다. 만약 그것이 실패하면 region mutator가 parse table을 만들고 각 symbol과 그에 상응하는 region 정보를 가져온다. 이제 우리는 어떤 symbol이 어떤 region에 속하는지 알 수 있고 그걸 통해 region-based swap, delete 함수를 만든다.  

~~~python
class RegionMutator(RegionMutator):
    def swap_fragment(self, seed):
        """Chooses a random region and swaps it with a fragment
           that starts with the same symbol"""
        if not seed.has_structure and seed.has_regions:
            regions = [r for r in seed.regions
                         if (len(seed.regions[r]) > 0 and
                            len(self.fragments[r]) > 0)]
            if len(regions) == 0: return seed

            key = random.choice(list(regions))
            s, e = random.choice(list(seed.regions[key]))
            swap_structure = random.choice(self.fragments[key])
            swap_string = tree_to_string(swap_structure)
            new_seed = Seed(seed.data[:s] + swap_string + seed.data[e:])
            new_seed.has_structure = False
            new_seed.has_regions = False
            return new_seed
        else:
            return super().swap_fragment(seed)

    def delete_fragment(self, seed):
        """Deletes a random region"""
        if not seed.has_structure and seed.has_regions:
            regions = [r for r in seed.regions
                         if len(seed.regions[r]) > 0]
            if len(regions) == 0: return seed

            key = random.choice(list(regions))
            s, e = (0, 0)
            while (e - s < 2):
                s, e = random.choice(list(seed.regions[key]))
            new_seed = Seed(seed.data[:s] + seed.data[e:])
            new_seed.has_structure = False
            new_seed.has_regions = False
            return new_seed
        else:
            return super().delete_fragment(seed)
~~~

parse table을 만들고 symbol들에 입력값의 region을 대입하는 데에는 Earley parser를 사용한다.
We can use the Earley parser to generate a parse table and assign regions in the input to symbols in the grammar. Our region mutators can substitute these region with fragments from the fragment pool that start with the same symbol, or delete these regions entirely.

### Focusing on Valid Seeds  
In the previous section, we have a problem: The low (degree of) validity. To address this problem, a validity-based power schedule assigns more energy to seeds that have a higher degree of validity. In other words, the fuzzer spends more time fuzzing seeds that are more valid.