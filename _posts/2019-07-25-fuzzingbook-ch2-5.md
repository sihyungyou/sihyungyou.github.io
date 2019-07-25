---
layout: post
title: "Software Testing Study CH 2-5"
tags: [Software Testing, Fuzzing, 공부]
comments: true
---

> Search-Based Fuzzing  

sometimes we are interested in deriving specific test input to reach specific statements in a program. To achieve such goal, we can search for it. Classic searching algorithm such as BFS or DFS wouldn't fit since they require us to check all the possible inputs. However, domain-knowledge can be used to overcome this problem. estimate which of inputs is closer to the one we are looking for -> guide us to reach to target. 

search based fuzzing example test_me(x, y) : x == 2 * (y + 1)를 만족하는 pair를 찾는 문제.  
~~~python
def test_me(x, y):
    if x == 2 * (y + 1):
        return True
    else:
        return False
~~~

이때 x, y의 neighbours 집합을 [x - 1, x, x + 1], [y - 1, y, y + 1]로 정의해둔다. 그리고 neighbours끼리의 pair를 하나씩 시도해보며 두 값의 차이(distance)가 0일 때 searching을 끝낸다.  
~~~python
def neighbours(x, y):
    return [(x + dx, y + dy) for dx in [-1, 0, 1]
            for dy in [-1, 0, 1]
            if (dx != 0 or dy != 0)
            and ((MIN <= x + dx <= MAX)
                 and (MIN <= y + dy <= MAX))]
~~~

이 알고리즘을 hillclimbing이라고 하는데 distance의 값이 점점 작아져서 0에 도달하는 과정을 그래프로 나타내면 언덕을 올라가는(실제로는 내려가는) 모양을 띠기 때문이다.  
~~~python
def hillclimber():
    # Create and evaluate starting point
    x, y = random.randint(MIN, MAX), random.randint(MIN, MAX)
    fitness = get_fitness(x, y)

    # Stop once we have found an optimal solution
    while fitness > 0:
        # Move to first neighbour with a better fitness
        for (nextx, nexty) in neighbours(x, y):
            new_fitness = get_fitness(nextx, nexty)

            # Smaller fitness values are better
            if new_fitness < fitness:
                x, y = nextx, nexty
                fitness = new_fitness
                break
~~~
하지만 이 알고리즘으로 모든 조건에 대해 searching을 성공할 수 있는 것은 아니다. test_me2(x, y)를 보자  
~~~python
def test_me2_instrumented(x, y):
    global distance
    distance = abs(x * x - y * y * (x % 20))
    if(x * x == y * y * (x % 20)):
        return True
    else:
        return False
~~~
이런 더 복잡한 조건을 hillclimbing 알고리즘으로 답을 찾으려 하면 반복이 끝나지 않을 수도 있다. neighbours 집합 내의 pair들이 모두 같거나 더 안좋은 fitness 값을 가질 수 있기 때문이다. 그 때 알고리즘이 머무는 점을 local optimum 이라고 하는데 여기서 벗어나기 위해서는 random restart를 하는 방법을 가장 쉽게 생각할 수 있다. hillclimbing 함수에서 fitness 값이 업데이트 되면 flag를 true로 바꾸고 매 반복에서 flag == false일 경우 restart를 해준다.  

지금까지는 x, y 값을 -1000 ~ 1000 범위의 작은 수로 제한시켰다. 이는 작은 수부터 시작해서 search하면 더 빨리 많은 테스트 케이스에 대해서 찾을 수 있기 때문이다. 하지만 찾으려는 답이 만약 search space와 완전히 다른곳에 있다면 어떨까. 작은 수로 시작한 hillclimber 알고리즘은 엄청나게 오래 걸릴 것이고 답을 찾을 가능성도 낮아질 것이다. 실제로 1000이 아니라 100000으로 범위를 확장하면 답을 찾는데 훨씬 많은 테스트케이스를 필요로 한다.  

### CGI Decoder as Search Problem  
CH2-2 에서 봤던 CGI Decoder 함수도 search-based fuzzing 개념을 적용할 수 있다. CGI Decoder는 문자열을 입력값으로 받는 함수인데 문자열에 속한 문자요소들도 숫자처럼 비교, 증가, 감소가 가능하기 때문이다. 예를 들어 입력값이 "test"일 때, 각 문자에 대해서 distance가 +1, -1인 모든 경우의 수를 나열하면 다음과 같다.  
~~~
uest
tfst
tett
tesu
sest
tdst
tert
tess
~~~

문자는 숫자와 동일하게 다뤄질 수 있으므로 위의 test_me() 예제에서 처럼 distance를 문자열에 대해서도 계산할 수 있다. 