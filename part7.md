---
id: part7
aliases: []
tags: []
---

The aim of this multi_part series is to annotate David Baezley's [Lambda Calculus from the Ground Up](https://www.youtube.com/watch?v=pkCLMl0e_0k)
Part6 focuses on the implementation of an `FACTORIAL` function, building on the implementation of `ISZERO` in [Part6](https://pavolkutaj.medium.com/the-annotated-lambda-calculus-from-the-ground-up-part-6-846182aff0e1)

It covers [02h30m11s](https://youtu.be/pkCLMl0e_0k?si=N03G39clne5pOEAu&t=9011)-[02h47m35s](https://youtu.be/pkCLMl0e_0k?si=q_Wp33yLeerg9Yx3&t=10055)

## 1. THE ASSEMBLY
* In the first 6 parts of the series/the workshop up until now, we have created an "assembly code" of functions

```
AND
 OR
NOT
SUCC
PRED
ADD
MUL
SUB
CONS
CAR
CDR
ISZERO
```

* ... it is kind of like **machine code**
* it is very abstract, purely in the world of function
* but having that - you can have more interesting things like writing **recursive functions**

## 2. FACTORIAL WITH RECURSION
* first, let's start with factorial in a python way

### 2.1. Usual Python implementation of factorial

```python
def fact(n):
    if n == 0:
        return 1
    else:
        return n * fact(n - 1)


assert fact(3) == 6
```

### 2.2. The new thing: take `ISZERO` to implement base case
* take the number
* do `ISZERO` check and return church boolean
* church booleans work like an if statement 

```python
def FACT(n: church_numeral) -> church_numeral:
  ISZERO(n) \
    (ONE) \
    (MUL(n)(FACT(PRED(n))))
```


#### 2.2.1. the build_up: for ISZERO, we need church_numerals + church_booleans

```python
ZERO = lambda fn: lambda init: init
assert ZERO(lambda n: n+1)(0) == 0
SUCC = lambda n: lambda fn: lambda init: fn(n(fn)(init))
ONE = SUCC(ZERO)
assert ONE(lambda n: n+1)(0) == 1
TRUE = lambda x: lambda y: x
assert TRUE(1)(0) == 1
FALSE = lambda x: lambda y: y
assert FALSE(1)(0) == 0
ISZERO = lambda n: n(lambda _: FALSE)(TRUE)
assert ISZERO(ZERO)(1)(0) == 1 # True is selecting left hand
assert ISZERO(ONE)(1)(0) == 0 # False is selecting right hand
# n is church_numeral -> we need church numerals
```

#### 2.2.2. the build_up: `MUL`:

```python
MUL = lambda x: lambda y: lambda f: y(x(f))
TWO = SUCC(ONE)
THREE = SUCC(TWO)
assert MUL(THREE)(TWO)(lambda n: n+1)(0) == 6
```

#### 2.2.3. the build_up: `PRED`

```python
CONS = lambda a: lambda b: lambda s: s(a)(b)
 CAR = lambda p: p(s=lambda a: lambda b: a)
 CDR = lambda p: p(s=lambda a: lambda b: b)
p=CONS(1)(0)
assert CONS(a=1)(b=0)(s=lambda a: lambda b: a) == 1
assert CAR(p) == 1
assert CDR(p) == 0
# ICONS = lambda n: CONS(SUCC(n))(n) - NOT THIS, THIS IS MY INTUITUION, maybe investigate
ICONS = lambda p: CONS(SUCC(CAR(p)))(CAR(p))
assert CAR(ICONS(CONS(ONE)(ZERO)))(lambda n: n+1)(0) == 2
PRED = lambda n: CDR(n(ICONS)(CONS(ZERO)(ZERO)))
assert PRED(SUCC(ONE))(lambda n: n+1)(0) == 1
```


#### 2.2.4. finished with the build_up ~> let's compose a first recursive function

```python
from typing import Callable, Any
church_numeral = Callable[..., Any]

def FACT(n: church_numeral) -> church_numeral: 
  ISZERO(n)(ONE)\
  (MUL(n)(FACT(PRED(n))))

TWO = SUCC(ONE)
THREE = SUCC(TWO)
assert THREE(lambda x: x+1)(0) == 3
assert FACT(THREE)(lambda x: x+1)(0) == 6
# RecursionError: maximum recursion depth exceeded
```

* it blows up in infinite recursion - why is that?
* increasing recursion depth is not going to help 

#### 2.2.5. explaining why the above is not working in python
does substition work in python same as it would in math/lisp


```python
def f(1+2): return 3*x+1
assert f(3) == 10
```

* so how does that work exactly
* it does the argument first and then it calls the function
* python evaluates arguments first: first `1+2` then pass into function
*

#### 2.2.6. this is python's eager evaluation
The problem seems to be the order of execution

```python
FACT = lambda n: ISZERO(n)(ONE)(MUL(n)(FACT(PRED(n))))
```

**Should be**: 
1. eval `ISZERO` **only** - once you get result which is `TRUE`/`FALSE`
2. eval `TRUE(ONE)(MUL(n)(FACT(PRED(n))))`

**Is**: everything is evaluated **eagerly**, then first function is evaluated. 

```python
from typing import Any


def LEFT():
    print("evaluated: left")
    return "return value: left"


def RIGHT():
    print("evaluated: right")
    return "return value: right"


def selector(choice) -> Any:
    print(f"selector {choice} called; only {choice} should be evaluated")
    if choice == "left":
        return lambda a: lambda b: a
    else:
        return lambda a: lambda b: b


print("Calling: selector(choice='left')(left())(right())")
result = selector(choice="left")(LEFT())(RIGHT())
result
```

which results

```
Calling: pick_left(left(), right())
LEFT evaluated!
RIGHT evaluated! # should NOT be evaluated! 
pick_left called, choosing first argument
Result: left_result
```

which means

```python
selector("left")(LEFT())(RIGHT())
#                  ↑       ↑
#                  |       |
#              Both are fully evaluated FIRST
#                  |       |
#                  ↓       ↓
selector("left")("return value: left")("return value: right")
#                           ↑                    ↑
#                           |                    |
#                    Now selector chooses between
#                    these pre-computed values
```

### 2.3. further explanation of python's eager eval
* the logic if the factorial can be - in python terms - expressed as

```python
def choose(t, a, b):
    if t:
        return a
    else:
        return b


a = 2
b = 3
assert choose(a < b, a, b) == 2
```

* but if `b` is supposed not to return, the whole logic collapses

```python
a=0
choose(t=a!=0, a=a, b=1/0 )
```

```
ZeroDivisionError                         Traceback (most recent call last)
Cell In[8], line 1
----> 1 choose(t=a!=0, a=a, b=1/0 )

ZeroDivisionError: division by zero

[ins] In [9]: 
```

* note that an `if` statement does not blow up like that

```python
a if a == 0 else 1/a
# >>> 0
```

* but the lambda calculus implementation of the `if` statement **does that**
* it does go down both paths, both the base case AND recurrence relation simultaneously
* this then explodes in infinite recursion

### 2.4. how can we turn that off the eager evaluation? move expression into a 0 arg function
* let's say we want to turn of `1/a`
* --> shove it in to 0 arg function, delay the calculation in some later point in time
* this is exactly how "function as parameters" work - they are called only when needed

```python
a = 0
f = lambda: 1 / a  # nothing happens
f()  # explodes upon call
# ZeroDivisionError: division by zero
```

### 2.5. apply the "trick" to the `choose` function

```python
from typing import Callable, Any


def choose(t: Any, a: Callable, b: Callable) -> Any:
    if t:
        return a()  # Evaluate only if needed
    else:
        return b()


a = lambda: 1 / 0
b = lambda: 1

choose((a == 0), a, b)
# 1
a = 0
choose(a == 0, lambda: a, lambda: 1 / a)
# 0
```

### 2.6. how to fix: implement lazy evaluation to the `FACT` function
introduce extra function call in there
it is not part of lambda calculus - it is hack to not to both branches of `if`
this concept of "when functions evaluate" is a big focus in functional programming
we are facing it here

### 2.7. apply the lambda wrapper to all of the elements of `FACT` what may break the functionality

```python
# numerals: zero + succ
ZERO = lambda f: lambda init: init
SUCC = lambda n: lambda f: lambda init: f(n(f)(init))
ONE = SUCC(ZERO)
assert ONE(f=lambda x: x+1)(init=1) == 2

# pairs
CONS = lambda x: lambda y: lambda s: s(x)(y)
 CAR = lambda p: p(lambda x: lambda y: x)
 CDR = lambda p: p(lambda x: lambda y: y)
assert CAR(CONS(1)(2)) == 1

# predecessor: numerals + pairs
ICONS = lambda p: CONS(SUCC(CAR(p)))(CAR(p))
assert CAR(ICONS(CONS(ONE)(ZERO)))(lambda n: n+1)(0) == 2 
PRED = lambda n: CDR(n(ICONS)(CONS(ZERO)(ZERO)))
assert PRED(ONE)(lambda n: n+1)(0) == 0

# lazy booleans
LAZY_TRUE = lambda x: lambda y: x()
LAZY_FALSE = lambda x: lambda y: y()

assert LAZY_TRUE(lambda: 1)(lambda: 0) == 1

# lazy ISZERO
ISZERO = lambda n: n(lambda f: LAZY_FALSE)(LAZY_TRUE)
assert ISZERO(ZERO)(lambda: 1)(lambda: 0) == 1 

# MUL
MUL = lambda x: lambda y: lambda f: y(x(f))
TWO = SUCC(ONE)
THREE = SUCC(TWO) 
assert MUL(THREE)(TWO)(lambda n: n+1)(0) == 6

# FINALLY - the lazy factorial
FACT = lambda n: ISZERO(n)\
        (lambda: ONE)\
        (lambda: MUL(n)(FACT(PRED(n))))

assert FACT(THREE)(lambda n: n+1)(0) == 6
```
