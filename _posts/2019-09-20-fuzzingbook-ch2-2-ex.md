---
layout: post
title: "Fuzzingbook Exercises CH 2-2"
tags: [Software Testing, Fuzzing, 공부, 한동대, 공프기]
comments: true
---

> Code Coverage  

### Exercise 1: Fixing cgi_decode  
Create an appropriate test to reproduce the `IndexError` discussed above.  Fix `cgi_decode()` to prevent the bug.  Show that your test (and additional `fuzzer()` runs) no longer expose the bug.  Do the same for the C variant.  

~~~python
while i < len(s):
    c = s[i]
    if c == '+':
        t += ' '
    elif c == '%' and(i+2 < len(s): # fixed here!
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
~~~
c == '%'의 경우 단순히 현재 가리키는 문자 뿐만 아니라 앞으로 최소 2개의 문자가 들어갈 공간이 있는지 또한 조건으로 추가하여 체크한다면 indexerror를 막을 수 있다.  

### Exercise 2 - Part 1: Compute branch coverage  
Define a function `branch_coverage()` that takes a trace and returns the set of pairs of subsequent lines in a trace – in the above example, this would be  

~~~python
set(
(('cgi_decode', 9), ('cgi_decode', 10)),
(('cgi_decode', 10), ('cgi_decode', 11)),
# more_pairs
)
~~~
