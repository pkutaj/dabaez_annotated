---
id: part6
aliases: []
tags: []
---

The aim of this multi_part series is to annotate David Baezley's [Lambda Calculus from the Ground Up](https://www.youtube.com/watch?v=pkCLMl0e_0k)
Part6 focuses on the implementation of an `ISZERO` statement, building on the implementation of subtraction in [Part5](https://medium.com/@pavolkutaj/the-annotated-lambda-calculus-from-the-ground-up-part-5-67a167c21808)
It covers [02h22m50s](https://youtu.be/pkCLMl0e_0k?si=scjnQ-BH_HVhFxTE&t=8570)-[02h30m-20s](https://youtu.be/pkCLMl0e_0k?si=znovWgVdq-X2nnpw&t=9020)

## 1. TEST FOR ZERO
* if you can test number for ZERO it allows you to start checking things
* why?
* if you can start doing tests, you can start thinking about control flow
* like if statement - test, and select one of two branches, essential for computability
* uncle bob reference to paper how can programing be reduced

### 1.1. test for zero: the build_up

```python
# build pairs
CONS = lambda a: lambda b: lambda s: s(a)(b)
CAR = lambda p: p(lambda a: lambda b: a)
CDR = lambda p: p(lambda a: lambda b: b)
assert CONS(1)(2)(lambda a: lambda b: a) == 1
assert CAR(CONS(1)(2)) == 1
assert CDR(CONS(1)(2)) == 2
# build church numerals with with successor function
ZERO = lambda fn: lambda init: init
assert ZERO(lambda n: n+1)(1) == 1
SUCC = lambda n: lambda fn: lambda init: fn(n(fn)(init))
 ONE = SUCC(ZERO)
 TWO = SUCC(ONE)
THREE = SUCC(TWO)
PLUS1= lambda n: n+1
assert ONE(PLUS1)(0) == 1
# build predecessor adding pairs to church numerals
ICONS = lambda p: CONS(SUCC(CAR(p)))(CAR(p))
PRED = lambda n: CDR(n(ICONS)(CONS(ZERO)(ZERO)))
assert PRED(THREE)(PLUS1)(0) == 2
# build subtract on the basis of predeccesor 
SUBTRACT = lambda x: lambda y: y(PRED)(x)
assert SUBTRACT(THREE)(ONE)(PLUS1)(0) == 2 # PASS
```

* we need one more thing to build for and this test and it is booleans

```python
TRUE = lambda left_hand: lambda right_hand: left_hand
FALSE = lambda left_hand: lambda right_hand: right_hand
assert TRUE(left_hand="green pill")(right_hand="red pill") == "green pill"
assert FALSE(left_hand="green pill")(right_hand="red pill") == "red pill"
```
### 1.2. test for zero: the addition
* since this is new, we first need a longhand
* the following works given our implementation of booleans as selectors between two "pills"
* the `ISZERO` contains only functions

```python
from typing import Callable, Any
church_numeral = Callable[..., Any]
church_boolean = Callable[..., Any]

def ISZERO(n: church_numeral) -> church_boolean: 
   return n(lambda _: FALSE)(TRUE)

assert ISZERO(ZERO)(left_hand=1)(right_hand=0) == 1 # PASS - TRUTH picks the left hand
assert ISZERO(ONE)(left_hand=1)(right_hand=0) == 0 # PASS - FALSE picks right hand
```

* the `lambda _: FALSE` is a **constant function** - it always returns `FALSE` if it called
* for `ZERO`, the `lambda g: FALSE` is never called, as the function is never called in a church numeral ZERO
* see https://realpython.com/python-double-underscore/ for underscores

```python
always_false = lambda g: False
[ins] In [2]: always_false(True)
Out[2]: False

[ins] In [3]: always_false(1)
Out[3]: False

[ins] In [4]: always_false("foobar")
Out[4]: False
```

* here is the trace using substitition rule from SICP

```
# Definitions for reference:
# TRUE = lambda x: lambda y: x
# FALSE = lambda x: lambda y: y
# ZERO = lambda fn: lambda init: init
# THREE = lambda f: lambda x: f(f(f(x)))
# ISZERO = lambda n: n(lambda g: FALSE)(TRUE)

--- Trace: ISZERO(ZERO) ---

ISZERO(ZERO)
=> (lambda n: n(lambda g: FALSE)(TRUE))(ZERO)
=> ZERO(lambda g: FALSE)(TRUE)
=> (lambda fn: lambda init: init)(lambda g: FALSE)(TRUE)
=> (lambda init: init)(TRUE)
=> TRUE

--- Trace: ISZERO(THREE) ---

ISZERO(THREE)
=> (lambda n: n(lambda g: FALSE)(TRUE))(THREE)
=> THREE(lambda g: FALSE)(TRUE)
=> (lambda f: lambda x: f(f(f(x))))(lambda g: FALSE)(TRUE)
=> (lambda x: (lambda g: FALSE)((lambda g: FALSE)((lambda g: FALSE)(x))))(TRUE)
=> (lambda g: FALSE)((lambda g: FALSE)((lambda g: FALSE)(TRUE)))
=> (lambda g: FALSE)((lambda g: FALSE)(FALSE)) # because (lambda g: FALSE)(TRUE) => FALSE
=> (lambda g: FALSE)(FALSE)                    # because (lambda g: FALSE)(FALSE) => FALSE
=> FALSE                                       # because (lambda g: FALSE)(FALSE) => FALSE
```

