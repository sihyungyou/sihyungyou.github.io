---
layout: post
title: "Software Testing Study CH 3-4"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Parsing Inputs  

Grammar를 다룬 이전 장들까지 문법이 어떻게 다양한 언어를 표현할 수 있는지 봤다. 또한 어떻게 문법이 문자열을 만드는데 사용되는지도 공부했다. Grammar는 그 반대도 가능하다. 문자열이 주어지면 그것을 이루는 구성요소(constituent parts)들로 쪼개는 것이다. 이 부분들은 추후에 다시 합쳐져서 새로운 문자열을 만들 수 있다.  

이번 장에서는 grammar를 주어진 valid seed input을 parse, decompose하는데 사용하여 각각에 대응하는 derivation tree를 만드는데 사용한다. 이것은 mutate, crossover, recombine을 할 수 있도록 해주고 이 세 가지는 궁극적으로 new valid, slightly changed inputs(fuzz)을 만들어 낼 수 있다.  

### Fuzzing a Simple Program  
간단한 csv 파일과 그것을 parse하여 문장 형태로 출력하는 코드가 있다.  
~~~python
def process_inventory(inventory):
    res = []
    for vehicle in inventory.split('\n'):
        ret = process_vehicle(vehicle)
        res.extend(ret)
    return '\n'.join(res)

# The CSV file contains details of one vehicle per line. Each row is processed in process_vehicle().
def process_vehicle(vehicle):
    year, kind, company, model, *_ = vehicle.split(',')
    if kind == 'van':
        return process_van(year, company, model)

    elif kind == 'car':
        return process_car(year, company, model)

    else:
        raise Exception('Invalid entry')

# Depending on the kind of vehicle, the processing changes.
def process_van(year, company, model):
    res = ["We have a %s %s van from %s vintage." % (company, model, year)]
    iyear = int(year)
    if iyear > 2010:
        res.append("It is a recent model!")
    else:
        res.append("It is an old but reliable model!")
    return res

def process_car(year, company, model):
    res = ["We have a %s %s car from %s vintage." % (company, model, year)]
    iyear = int(year)
    if iyear > 2016:
        res.append("It is a recent model!")
    else:
        res.append("It is an old but reliable model!")
    return res
~~~

~~~python
mystring = """\
1997,van,Ford,E350
2000,car,Mercury,Cougar\
"""
print(process_inventory(mystring))
~~~
~~~
We have a Ford E350 van from 1997 vintage.
It is an old but reliable model!
We have a Mercury Cougar car from 2000 vintage.
It is an old but reliable model!
~~~

이 프로그램을 fuzz해보자. grammar-based fuzzer를 다루고 있으므로 csv 파일에 대한 문법을 작성할 수 있다.  
~~~python
CSV_GRAMMAR = {
    '<start>': ['<csvline>'],
    '<csvline>': ['<items>'],
    '<items>': ['<item>,<items>', '<item>'],
    '<item>': ['<letters>'],
    '<letters>': ['<letter><letters>', '<letter>'],
    '<letter>': list(string.ascii_letters + string.digits + string.punctuation + ' \t\n')
}
~~~

그리고 1000개의 값을 만들어서 process_vehicle() 의 결과를 보면 (당연히) 그 어떤 입력값도 제대로 프로그램에 적용되지 않는다. 이유는 단순하다. "Invalid Entry" 현재 사용 중인 GrammarFuzzer는 위에서 작성한 CSV_GRAMMAR를 이해하고 있지 않기 때문이다. 설사 CSV_GRAMMAR를 이해한다고 해도 입력값이 그에 맞는 format을 갖추지 않으면 높은 확률로 쓸모없는 값이 나올 것이다. 그렇다면 실제 template과 valid value를 샘플에서 뽑아온 후 그것을 fuzzing하는 데에 사용하면 어떨까? parser를 쓰면 가능하다.  

### Using a Parser  
Parser는 입력을 처리하는 프로그램이다 (general meaning) 이번 장에서 다룰 parser는 input string을 derivation tree로 바꾸는 프로그램을 의미한다. 유저의 입장에서 두 가지 단계를 거쳐서 그것을 수행한다.  
1. parser를 정의한 grammar로 initialize (grammar 이해시키기)  
2. parser 사용 (derivation tree로 변환)  
변환이 끝나면 그대로 grammar fuzzing에 의해 만들어진 tree로써 사용하면 된다.  

### An Ad Hoc Parser  
위의 예시에서 보았듯 프로그래머는 어떤 규칙을 가진 data로부터 특정 부분을 뽑아와야 할 때가 있다. 그리고 그 규칙을 프로그래머가 하나하나 이해하고 parser에 더할 부분이 있으면 고친다. 규칙이 늘어날수록, 재귀적인 문법이 필요할수록 (json format) 점점 어려워 진다. 하지만 입력될 문자열들이 이미 정의된, 통일된 문법을 가지고 있다면 parser는 그 문법을 이해하면 된다.  

### Grammar  
다양한 데이터에 다양한 문법이 있고, 각 문법마다 단순히 parser로 해결하기 쉽지 않은 문제가 있다.  
먼저 언급했듯, 다양한 문법이 있다는 것 자체만으로도 벌써 문제다.  
그리고 handle 하기 꽤나 까다로운 recursion이 있다.  
마지막으로 ambiguity 문제가 있다. 예를 들어 1+2+3은 우리에게 직관적이지만 컴퓨터 입장에서 derivation tree를 만들 때 (1+2)+3인지, 1+(2+3)인지 분명히 구분해야 할 것이다.  
