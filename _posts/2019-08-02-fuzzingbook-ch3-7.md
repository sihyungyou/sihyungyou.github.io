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
In this book, let us stay with HTML input for a while. To generate valid HTML inputs for our Python HTMLParser, we should first define a simple grammar. It allows to define HTML tags with attributes. Our context-free grammar does not require that opening and closing tags must match. However, we will see that such context-sensitive features can be maintained in the derived input fragments, and thus in the generated inputs.  

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
We are now ready to implement our first input-structure-aware mutator. Let's initialize the mutator with the dictionary fragments representing the empty fragment pool. It contains a key for each symbol in the grammar (and the empty set as value).  

The FragmentMutator adds fragments recursively. A fragment is a subtree in the parse tree and consists of the symbol of the current node and child nodes (i.e., descendant fragments). We can exclude fragments starting with symbols that are tokens, terminals, or not part of the grammar.  

The function add_to_fragment_pool() parses a seed (no longer than 200ms) and adds all its fragments to the fragment pool. If the parsing the  seed was successful, the attribute seed.has_structure is set to True. Otherwise, it is set to False.  

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

For many symbols in the grammar, we have collected a number of fragments. There are several open and closing tags and several interesting fragments starting with the xml-tree symbol.

Summary. For each interesting symbol in the grammar, the FragmentMutator has a set of fragments. These fragments are extracted by first parsing the inputs to be mutated.

### Fragment-Based Mutation  
We can use the fragments in the fragment pool to generate new inputs. Every seed that is being mutated is disassembled into fragments, and memoized – i.e., disassembled only the first time around.  

Our first structural mutation operator is swap_fragments(), which choses a random fragment in the given seed and substitutes it with a random fragment from the pool. We make sure that both fragments start with the same symbol. For instance, we may swap a closing tag in the seed HTML by another closing tag from the fragment pool.

In order to choose a random fragment, the mutator counts all fragments (n_count) below the root fragment associated with the start-symbol.

Our structural mutator chooses a random number between 2 (i.e., excluding the start symbol) and the total number of fragments (n_count) and uses the recursive swapping to generate the new fragment. The new fragment is serialized as string and returned as new seed.  
(We can use a similar recursive traversal to remove a random fragment.)
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

Summary. We now have all ingredients for structure-aware fuzzing. Our mutator disassembles all seeds into fragments, which are then added to the fragment pool. Our mutator swaps random fragments in a given seed with fragments of the same type. And our mutator deletes random fragments in a given seed. This allows to maintain a high degree of validity for the generated inputs w.r.t. the given grammar.  

### Integration with Greybox Fuzzing  
Our fragment-level blackbox fuzzer (LangFuzzer) generates more valid inputs but achieves less code coverage than a fuzzer with our byte-level fuzzer. So, there is some value in generating inputs that do not stick to the provided grammar.

In the following we integrate fragment-level blackbox fuzzing (LangFuzz-style) with byte-level greybox fuzzing (AFL-style). The additional coverage-feedback might allow us to increase code coverage more quickly.

A greybox fuzzer adds to the seed population all generated inputs which increase code coverage. Inputs are generated in two stages, stacking up to four structural mutations and up to 32 byte-level mutations.

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
Our structural greybox fuzzer  
1. runs faster than the fragment-based LangFuzzer,  
2. achieves more coverage than both the fragment-based LangFuzzer and the vanilla blackbox mutational fuzzer, and  
3. generates fewer valid inputs than even the vanilla blackbox mutational fuzzer.  

### Mutating Invalid Seeds  
In the previous section, we have seen that most inputs that are added as seeds are invalid w.r.t. our given grammar. Yet, in order to apply our fragment-based mutators, we need it to parse the seed successfully. Otherwise, the entire fragment-based approach becomes useless. The question arises: How can we derive structure from (invalid) seeds that cannot be parsed successfully?

To this end, we introduce the idea of region-based mutation, first explored with the AFLSmart structural greybox fuzzer [Van-Thuan Pham et al, 2018.]. AFLSmart implements byte-level, fragment-based, and region-based mutation as well as validity-based power schedules. We define region-based mutators, where a region is a consecutive sequence of bytes in the input that can be associated with a symbol in the grammar.

### Determining Symbol Regions  
The function chart_parse of the Earley parser produces a parse table for a string. For each letter in the string, this table gives the potential symbol and a region of neighboring letters that might belong to the same symbol. Unlike input fragments, input regions can be derived even if the parser fails to generate the entire parse tree.

### Region-based Mutation  
To fuzz invalid seeds, the region-based mutator associates symbols from the grammar with regions (i.e., indexed substrings) in the seed. The overridden method add_to_fragment_pool() first tries to mine the fragments from the seed. If this fails, the region mutator uses Earley parser to derive the parse table. For each column (i.e., letter), it extracts the symbols and corresponding regions. This allows the mutator to store the set of regions with each symbol.

Now that we know which regions in the seed belong to which symbol, we can define region-based swap and delete operators.

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

We can use the Earley parser to generate a parse table and assign regions in the input to symbols in the grammar. Our region mutators can substitute these region with fragments from the fragment pool that start with the same symbol, or delete these regions entirely.

### Focusing on Valid Seeds  
In the previous section, we have a problem: The low (degree of) validity. To address this problem, a validity-based power schedule assigns more energy to seeds that have a higher degree of validity. In other words, the fuzzer spends more time fuzzing seeds that are more valid.