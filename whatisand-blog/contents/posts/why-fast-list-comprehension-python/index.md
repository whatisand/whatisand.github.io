---
title: "파이썬에서 List Comprehension이 더 빠른 이유"
description: "for loop보다 list comprehension이 더 빠른건 알지만 왜 빠른지 궁금하다면"
date: 2022-02-27
update: 2022-02-27
tags:
  - python
series: "오늘의 개발일기"
---

## 들어가며
보통 언어를 공부하면 쉽게 사용할 수 있도록 만들어진 문법일수록 실행시간에서 손해보는 경우가 많았습니다.
리스트 컴프리헨션도 일종의 언어 수준의 편의기능으로 생각했기에 당연히 느릴 것이라고 생각했습니다. 언어의 편의기능을 사용하면 가독성과 개발속도에서 이점을 가져가고 속도에서 약간 손해를 보는 것이라고 생각하면 이해가 쉬웠습니다.
그런데 자세히 알게 되니 충격적인 사실이 있었습니다. 리스트 컴프리헨션을 쓰는 이유는 속도측면도 있다는 사실을 알았습니다.
왜 그럴까? 왜 그렇게 만들어졌을까? 궁금해서 찾아보았습니다.


## 접근 방법
많은 분들이 리스트 컴프리헨션을 설명해주셨지만 제 마음에 드는 설명이 없었습니다. 리스트 컴프리헨션이 더 빠르다는 것도 충분히 중급 이상의 지식으로 취급되다 보니 더욱 자세한 설명을 찾기는 쉽지 않았습니다.

사실 이 글을 쓰기 무색하게도 이미 제가 접근하고 싶은 방법대로 똑같이 접근해주신 분이 계셨습니다. 욱재님께 감사드립니다.

파이썬은 스크립트 언어이지만 실행하기 전에 자체 엔진에서 해석할 수 있는 바이너리 코드로 변환(컴파일)됩니다. 동작은 같지만 바이너리 코드 상에서 다를 수 있기 때문에 성능을 비교할 때 바이너리 코드를 확인하곤 합니다.

파이썬에서는 아래 모듈을 통해 바이너리 코드를 쉽게 읽을 수 있는 형태로 볼 수 있습니다.

``` python
>>> import dis
>>> def hello_world():
...     print("hello world!")
...
>>> dis.dis(hello_world)
  2           0 LOAD_GLOBAL              0 (print)
              2 LOAD_CONST               1 ('hello world!')
              4 CALL_FUNCTION            1
              6 POP_TOP
              8 LOAD_CONST               0 (None)
             10 RETURN_VALUE
>>>
```

이 모듈을 활용하여 for loop로 작동할때와, 같은 동작을 list comprehension으로 구현했을때의 바이너리 코드를 분석하여 작동을 비교해보려고 합니다.

모듈에서 나타난 바이트 코드는 [dis — Disassembler for Python bytecode — Python 3.7.12 documentation](https://docs.python.org/3.7/library/dis.html#opcode-LIST_APPEND) 에 있습니다.


## 작동 방식 

#### List append 를 사용할 때의 바이트 코드
``` python
>>> import dis
>>> def use_append():
...     result = []
...     for i in range(10000):
...             result.append(i)
...     return result
...
>>> dis.dis(use_append)
  2           0 BUILD_LIST               0
              2 STORE_FAST               0 (result)

  3           4 LOAD_GLOBAL              0 (range)
              6 LOAD_CONST               1 (10000)
              8 CALL_FUNCTION            1
             10 GET_ITER
        >>   12 FOR_ITER                14 (to 28)
             14 STORE_FAST               1 (i)

  4          16 LOAD_FAST                0 (result)
             18 LOAD_METHOD              1 (append)
             20 LOAD_FAST                1 (i)
             22 CALL_METHOD              1
             24 POP_TOP
             26 JUMP_ABSOLUTE           12

  5     >>   28 LOAD_FAST                0 (result)
             30 RETURN_VALUE

```




#### List Comprehension 의 바이트 코드
``` python
>>> def use_compre():
...     return [i for i in range(10000)]
...
>>> dis.dis(use_compre)
  2           0 LOAD_CONST               1 (<code object <listcomp> at 0x7fe36818f870, file "<stdin>", line 2>)
              2 LOAD_CONST               2 ('use_compre.<locals>.<listcomp>')
              4 MAKE_FUNCTION            0
              6 LOAD_GLOBAL              0 (range)
              8 LOAD_CONST               3 (10000)
             10 CALL_FUNCTION            1
             12 GET_ITER
             14 CALL_FUNCTION            1
             16 RETURN_VALUE

Disassembly of <code object <listcomp> at 0x7fe36818f870, file "<stdin>", line 2>:
  2           0 BUILD_LIST               0
              2 LOAD_FAST                0 (.0)
        >>    4 FOR_ITER                 8 (to 14)
              6 STORE_FAST               1 (i)
              8 LOAD_FAST                1 (i)
             10 LIST_APPEND              2
             12 JUMP_ABSOLUTE            4
        >>   14 RETURN_VALUE
```

두 코드의 차이점을 보겠습니다.

리스트 컴프리헨션을 사용할 때의 코드를 보면 두 부분으로 나누어져 있는 것을 확인할 수 있습니다. MAKE_FUNCTION을 보니 자체적으로 함수를 만들어서 실행하는 것 같습니다. <listcomp>라고 하는 오브젝트가 있습니다.


우리는 for loop를 이용한 것과 list comprehension을 이용한 것을 비교하고는 합니다. 루프에서 어떤 차이가 있는지 보겠습니다.

for loop를 이용한 함수에서는 12번 부터 26번 전까지 loop를 수행합니다.
list comprehension 에서는 <listcomp> 아래 4번부터 12번까지 loop를 수행합니다.

for loop 에서는 append method를 call 하고, lists comprehension에서는 LIST_APPEND 바이트 코드를 사용하여 append를 하는 것으로 추측됩니다.

`LIST_APPEND` 를 사용하는 것과 append를 `CALL_METHOD`로 불러오는 것의 차이때문으로 추측해볼 수 있는데, 실제로 문서에 따르면 LIST_APPEND에서 불러오는 코드와 list append에서 불러오는 함수는 동일한 함수를 불러오고 있습니다.

결국 두 방식 모두 append하는 부분에서는 차이가 없습니다.

## 실제로 차이가 나는 부분은?
list append 방식에서는 `CALL_METHOD`를 통해 append 메소드를 사용합니다.
그렇다면 `CALL_METHOD` 로직을 타면서 생기는 비효율이 있지 않을까 생각해볼 수 있습니다.

욱재님의 블로그에서 빈 함수를 불러오는 for loop와 아무 것도 실행되지 않는 for loop를 도는 시간차이를 비교해 두었습니다. 저도 한번 해보겠습니다.

``` python
% python3 -m timeit -s "def empty(): pass" "for i in range(10000000): empty()"
1 loop, best of 5: 665 msec per loop

% python3 -m timeit "for i in range(10000000): pass"
2 loops, best of 5: 189 msec per loop
```

진짜로 엄청난 시간 차이가 발생합니다.

list append 메소드를 사용하는 것과, list comprehension에서 자체적으로 append를 하는 것은 파이썬이 특정 메소드를 불러올 때 생기는 오버해드에서 생겨난 것이라고 할 수 있습니다.


## 참고자료
- [🐍 List Comprehension이 빠른 이유를 찾아보자 – Ukjae Jeong](https://jeongukjae.github.io/posts/inspecting-list-comprehension/)