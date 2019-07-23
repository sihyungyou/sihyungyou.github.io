---
layout: post
title: "Software Testing Study CH 2-2"
tags: [Software Testing, Fuzzing, 공부]
comments: true
---

> Code Coverage  

이 장에서는 Software Teting 의 효율을 어떻게 측정하는 지에 대해서 다룬다. Coverage라는 개념을 소개하는데 지난 장에서 이야기한 basic fuzzing의 case에서 버그가 적거나 충분치 않게 검출되었을 때 유용하게 쓰이는 개념이다. Coverage는 프로그램의 어떤 부분이 테스팅 중에 실제로 실행되었는지 파악하고 실행된 함수들의 위치(line number) 정보까지 프로그래머에게 전달한다. 이를 설명하기 위해 CGI Decoder(공백이 들어가면 안되고 16진수를 %xx로 표시하도록 되어있는 등 규칙이 있는 URL을 decode 하는 함수)로 demo를 진행한다.  

~~~python
def cgi_decode(s):
    """Decode the CGI-encoded string `s`:
       * replace "+" by " "
       * replace "%xx" by the character with hex number xx.
       Return the decoded string.  Raise `ValueError` for invalid inputs."""

    # Mapping of hex digits to their integer values
    hex_values = {
        '0': 0, '1': 1, '2': 2, '3': 3, '4': 4,
        '5': 5, '6': 6, '7': 7, '8': 8, '9': 9,
        'a': 10, 'b': 11, 'c': 12, 'd': 13, 'e': 14, 'f': 15,
        'A': 10, 'B': 11, 'C': 12, 'D': 13, 'E': 14, 'F': 15,
    }

    t = ""
    i = 0
    while i < len(s):
        c = s[i]
        if c == '+':
            t += ' '
        elif c == '%':
            digit_high, digit_low = s[i + 1], s[i + 2]
            i += 2
            if digit_high in hex_values and digit_low in hex_values:
                v = hex_values[digit_high] * 16 + hex_values[digit_low]
                t += chr(v)
            else:
                raise ValueError("Invalid encoding")
        else:
            t += c
        i += 1
    return t
~~~

예를 들어 cgi_decode("Hello+world") 의 출력값은 'Hello world' 이다.  

### Black-Box Testing  
Black-Box Testing은 specification에 의존하는, 특정 경우들에 대해서만 테스팅을 하는 개념이다. 즉 위의 cgi_decode()를 예로 들자면 +. %xx, 일반 문자열, 혹은 입력되면 안되는 문자일 경우 네 가지 케이스에 대해서 cover 가능 하다. 장점은 specified behavior에서 에러를 찾기에 유용하다는 것이다. 단, 모든 구현에 대해서 커버하지는 못한다. Black-Box 라고 불리는 이유는 말 그대로 검정 상자이기에 코드 내부를 보고 그 구현에 대한 error detecting이 불가하기 때문이다. 그래서 각 input (precondition)과 output (postcondition)의 특정한 behavior를 확인하며 testing을 하는 것이다.  

### White-Box Texting  
White-Box Testing은 implementation에 대해 테스팅하는 개념이다. 구현된 모든 statment에 대해서 검사를 하는데 테스팅 중에 execute 되지 않는 코드는 에러가 trigger될 가능성도 없다. cgi_decode()를 예로 들면 black-box testing과 달리 모든 경우에 대해 입력값이 대응되어야 하는데 %문자 다음에 반드시 16진수 숫자가 들어오지 않을 수도 있으므로 그 경우에 대해서도 고려한다. Black-Box와 달리 투명하게 모두 볼 수 있다고 하여 White-Box testing이라고 불리며 내부의 코드를 보고 에러를 찾는 테스팅이다. 아래의 두 criteria가 가장 흔히 쓰인다.   

- Statement coverage : each statement in the code must be executed by at least one test input.  
- Branch coverage : each branch in the code must be taken by at least one test input. (This translates to each if and while decision once being true, and once being false.)  

### Tracing Executions  
White-box testing의 장점중 하나는 프로그램의 어떤 부분이 cover 되었는지 자동적으로 판단할 수 있다는 것이다. 이를 위해서는 각 executed/not executed codes의 정보를 프로그래머에게 전달해줘야 한다. 그 정보를 모으는 것이 tracing이라고 할 수 있다. 테스트 하려는 함수이름을 주면 몇 번째 line에서 실행되었는지 반환하거나, 프로그램의 입력값에 대해 그것을 처리하기 위해서 어떤 함수들이 실행되었는지 line no를 반환하기도 한다. 적절하게 tracing 함수를 adjust 함으로써 coverage 끼리 비교하는 것도 가능하다. 예를 들면 두 다른 문자열에 대해 coverage를 기록하고 차집합을 출력하면 한 문자열에서만 실행된 코드 라인을 알아낼 수 있다. 혹은 모든 경우에 대해 기록한 coverage에서 특정 문자열의 coverage를 빼면 실행되지 않은 함수를 알아낼 수도 있다.  

### Coverage Class  

~~~python
class Coverage(object):
    # Trace function
    def traceit(self, frame, event, arg):
        if self.original_trace_function is not None:
            self.original_trace_function(frame, event, arg)

        if event == "line":
            function_name = frame.f_code.co_name
            lineno = frame.f_lineno
            self._trace.append((function_name, lineno))

        return self.traceit

    def __init__(self):
        self._trace = []

    # Start of `with` block
    def __enter__(self):
        self.original_trace_function = sys.gettrace()
        sys.settrace(self.traceit)
        return self

    # End of `with` block
    def __exit__(self, exc_type, exc_value, tb):
        sys.settrace(self.original_trace_function)

    def trace(self):
        """The list of executed lines, as (function_name, line_number) pairs"""
        return self._trace

    def coverage(self):
        """The set of executed lines, as (function_name, line_number) pairs"""
        return set(self.trace())
~~~

코드 자체는 어렵지 않은 알고리즘이다. Coverage 클래스는 실행된 코드에 대해서 line number, function name 모두를 기록한다.  

### Finding Errors with Basic Fuzzing  
위의 tracing 과정에서 fuzzing의 개념을 추가하면 random, massive한 입력값에 대해 coverage를 기록하면서 버그를 검출하는 것으로 개념을 확장시킬 수 있다. 예를 들면 문자열의 끝에 %가 오는 경우는 뒤에 두 자릿수 숫자 대신 널 문자가 자리한다. digit high, low를 검사하는데 문자열의 범위를 벗어난 memory를 접근하게 되는 것이다. 이와 같이 fuzzing을 통해 error finding의 가능성을 높일 수 있다.  

### 배운 점  
- Difference between black-Box and white-Box testing  
- Measuring the effectiveness of software testing via coverage  