---
layout: post
title: "Software Testing Study CH 2-4 (2)"
tags: [Software Testing, Fuzzing, 공부]
comments: true
---

> Greybox Fuzzing  

### Directed Greybox Fuzzing  
가끔은 buffer overflow를 기대하고 fuzzer가 위험한 메모리 영역을 접근하기를 바랄 때도 있다. 혹은 가장 최근에 수정된 코드에 대해서 테스팅을 해보고 싶을 수도 있다. 어떻게 그러한 특정 영역으로 fuzzer를 가이드 할 수 있을까. 이번 장에서는 directed greybox fuzzer를 통해 이 주제에 대해서 알아본다.  

### Solving a Maze  
문자열 미로를 예로 들어보자. 시작점에서 끝점까지 미로를 푸는 문자열을 입력으로 준다.  
D = down, L = left, R = right, U = up 이다.  

~~~python
maze_string = """
+-+-----+
|X|     |
| | --+ |
| |   | |
| +-- | |
|     |#|
+-----+-+
"""
~~~

~~~python
print(maze("DDDDRRRRUULLUURRRRDDDD"))
~~~

~~~
SOLVED

+-+-----+
| |     |
| | --+ |
| |   | |
| +-- | |
|     |X|
+-----+-+
~~~

maze string의 각 문자는 tile이고 그 의미는 다음과 같다.  
If the current tile is "benign" (), the tile-function corresponding to the next input character (D, U, L, R) is called. Unexpected input characters are ignored. If no more input characters are left, it returns "VALID" and the current maze state.  
If the current tile is a "trap" (+,|,-), it returns "INVALID" and the current maze state.  
If the current tile is the "target" (#), it returns "SOLVED" and the current maze state.  

### 접근  
아직 fuzzing과 미로가 무슨관련인가 싶다. 그 접근을 천천히 살펴보자. DictMutator class는 문자열을 주어진 dictionary에서 keyword를 뽑아 삽입하는 방식으로 mutate하는 class다. 미로를 fuzz 하기 위해서 이 클래스를 keyword를 seed의 끝에 append하고 seed의 마지막 문자를 삭제하는 방식으로 확장시킨다.  

~~~python
class MazeMutator(DictMutator):
    def __init__(self, dictionary):
        super().__init__(dictionary)
        self.mutators.append(self.delete_last_character)
        self.mutators.append(self.append_from_dictionary)

    def append_from_dictionary(self,s):
        """Returns s with a keyword from the dictionary appended"""
        random_keyword = random.choice(self.dictionary)
        return s + random_keyword
    
    def delete_last_character(self,s):
        """Returns s without the last character"""
        if (len(s) > 0):
            return s[:-1]
~~~

classic power schedule, extended maze mutator를 적용시켜서 greybox fuzzer를 시도해보자. 입력 문자열을 fuzz해서 미로를 푸는 것이 목적이다. 당연히 fuzz의 대상인 population은 L, R, U, D이다.  

~~~python
n = 10000
seed_input = " " # empty seed

maze_mutator = MazeMutator(["L","R","U","D"])
maze_schedule = PowerSchedule()
maze_fuzzer  = GreyboxFuzzer([seed_input], maze_mutator, maze_schedule)

print_stats(maze_fuzzer)
~~~
~~~
Out of 888 seeds, 
*    0 solved the maze, 
*  184 were valid but did not solve the maze, and 
*  704 were invalid
~~~

### Computing Function-Level Distance  
(이 부분 아직 이해 부족크..)  
static call graph를 이용하면 함수 f와 타겟 t사이의 거리를 계산할 수 있다. 그리고 그것을 기반으로 call graph에 상응하는 function을 찾아야 한다. distance dictionary는 각 function과 target까지의 거리 정보를 담고 있다. 만약 target 까지 길이 없다면 최댓값으로 거리를 조정한다.  

### Directed Power Schedule  
위의 function-level distance 계산을 마쳤다면 power schedule algorithm을 다시한번 고쳐본다. lower average distance to target function일 수록 더 높은 seed energy를 주는 것이다.  
~~~python
class DirectedSchedule(PowerSchedule):
    def __init__(self, distance, exponent):
        self.distance = distance
        self.exponent = exponent

    def __getFunctions__(self, coverage):
        functions = set()
        for f, _ in set(coverage):
            functions.add(f)
        return functions
    
    def assignEnergy(self, population):
        """Assigns each seed energy inversely proportional
           to the average function-level distance to target."""
        for seed in population:
            if not hasattr(seed, 'distance'):
                num_dist = 0
                sum_dist = 0
                for f in self.__getFunctions__(seed.coverage):
                    if f in list(distance):
                        sum_dist += distance[f]
                        num_dist += 1
                seed.distance = sum_dist / num_dist
                seed.energy = (1 / seed.distance) ** self.exponent
~~~

### Improved Directed Power Schedule  
directed power schedule을 한 번 더 개선시켜보자. 최소, 최대 사이의 거리로 seed distance를 normalize 시키는 방법을 이용한다. 
~~~python
class AFLGoSchedule(DirectedSchedule):
    def assignEnergy(self, population):
        """Assigns each seed energy inversely proportional
           to the average function-level distance to target."""
        min_dist = 0xFFFF
        max_dist = 0
        for seed in population:
            if not hasattr(seed, 'distance'):
                num_dist = 0
                sum_dist = 0
                for f in self.__getFunctions__(seed.coverage):
                    if f in list(distance):
                        sum_dist += distance[f]
                        num_dist += 1
                seed.distance = sum_dist / num_dist
            if seed.distance < min_dist: min_dist = seed.distance
            if seed.distance > max_dist: max_dist = seed.distance

        for seed in population:
            if (seed.distance == min_dist):
                if min_dist == max_dist:
                    seed.energy = 1
                else: 
                    seed.energy = max_dist - min_dist
            else:
                seed.energy = ((max_dist - min_dist) / (seed.distance - min_dist)) 
~~~

### 배운 점  
- blackbox mutation-based fuzzer는 mutator와 power schedule 개념을 합쳐서 population으로부터 입력값을 랜덤하게 선택해와 mutation을 적용한다.  
- greybox mutation-based fuzzer는 coverage information을 활용하는데, seed population에 coverage를 증가시키는 입력값을 더해가는 방식으로 작동한다.  
- boosted greybox fuzzer는 더 많은 coverage를 약속하는 seed에게 더 많은 energy를 주는 power schedule algorithm을 사용한다. 더 많은 coverage란 unusual paths를 더 많이 학습하는 코드를 의미한다.  
- directed greybox fuzzer는 unusual path 보다는 target 까지 더 가까운 함수를 제공하는 seed에게 더 많은 energy를 준다.