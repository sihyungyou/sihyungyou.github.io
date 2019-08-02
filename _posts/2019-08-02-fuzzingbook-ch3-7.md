---
layout: post
title: "Software Testing Study CH 3-7"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Greybox Fuzzing with Grammars  

In this chapter, we introduce two important extensions to our syntactic fuzzing techniques:

We show how to combine parsing and fuzzing with grammars. This allows to mutate existing inputs while preserving syntactical correctness, and to reuse fragments from existing inputs while generating new ones. The combination of parsing and fuzzing, as demonstrated in this chapter, has been highly successful in practice: The LangFuzz fuzzer for JavaScript has found more than 2,600 bugs in JavaScript interpreters this way.

In the previous chapters, we have used grammars in a black-box manner – that is, we have used them to generate inputs regardless of the program being tested. In this chapter, we introduce mutational greybox fuzzing with grammars: Techniques that make use of feedback from the program under test to guide test generations towards specific goals. As in lexical greybox fuzzing, this feedback is mostly coverage, allowing us to direct grammar-based testing towards uncovered code parts.

### Background  
First, we recall a few basic ingredients for mutational fuzzers.

Seed. A seed is an input that is used by the fuzzer to generate new inputs by applying a sequence of mutations.
Mutator. A mutator implements a set of mutation operators that applied to an input produce a slightly modified input.
PowerSchedule. A power schedule assigns energy to a seed. A seed with higher energy is fuzzed more often throughout the fuzzing campaign.
MutationFuzzer. Our mutational blackbox fuzzer generates inputs by mutating seeds in an initial population of inputs.
GreyboxFuzzer. Our greybox fuzzer dynamically adds inputs to the population of seeds that increased coverage.
FunctionCoverageRunner. Our function coverage runner collects coverage information for the execution of a given Python function.

### Building a Keyword Dictionary  
To fuzz our HTML parser, it may be useful to inform a mutational fuzzer about important keywords in the input – that is, important HTML keywords. To this end, we extend our mutator to consider keywords from a dictionary.
~~~python
dict_mutator = DictMutator(["<a>","</a>","<a/>", "='a'"])
~~~

![Center example image](https://user-images.githubusercontent.com/35067611/62351047-70b52e80-b53f-11e9-9cc3-c32ac8a15e3c.png "Center"){: .center-image}  

Informing the fuzzer about important keywords already goes a long way towards achieving lots of coverage quickly.

### Fuzzing with Input Fragments  
While dictionaries are helpful to inject important keywords into seed inputs, they do not allow to maintain the structural integrity of the generated inputs. Instead, we need to make the fuzzer aware of the input structure. We can do this using grammars. Our first approach  

1. parses the seed inputs,  
2. disassembles them into input fragments, and  
3. generates new files by reassembling these fragments according to the rules of the grammar.

This combination of parsing and fuzzing can be very powerful, as we will see in an instant  

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

### Fragment-Based Fuzzing  
We can now define a input-structure aware fuzzer as pioneered in LangFuzzer. To implement LangFuzz, we modify our blackbox mutational fuzzer to stack up to four structural mutations.  
