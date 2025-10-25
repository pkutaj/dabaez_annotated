---
id: part2
aliases: []
tags: []
---

The aim of this multi_part series is to annotate David Baezley's [Lambda Calculus from the Ground Up](https://www.youtube.com/watch?v=pkCLMl0e_0k)
This, part2, is about numbers + arithmetics expressed with lambda calculus. 
It is inspired by [PeanoArithmetic](https://en.wikipedia.org/wiki/Peano_axioms).
Timewise, this part covers ~ 0h:35min to 1h:08min of the lecture. Use the link above to start watching. Use Python Repl to follow. 
StartURL: https://youtu.be/pkCLMl0e_0k?si=G2AZCYnjV3Wlq02M&t=2083

## 1. NUMBERS
* do not overthink numbers - go back to kindergarten and use fingercounting and 
* visual representation
* how can we do that with numbers?
* you can have infinite number of function calls

```python
one = lambda f: lambda x: f(x)
two = lambda f: lambda x: f(f(x))
three = lambda f: lambda x: f(f(f(x)))
```

* `x` is the starting point
* `f` is the function that is being repeated

```python
def two(f):
    def init(x):
        return f(f(x))

    return init


assert two(lambda n: n + 1)(0) == 2
```

* let me rewrite this

### 1.1. the `init` name is incorrect - the point of `init` is to apply the `action` from the first param

```python
# the abstraction of two with lambdas with f as first param and x as another param
two = lambda f: lambda x: f(f(x))

# the abstraction of two with named functions
from typing import any


def two(action: callable) -> callable:
    def apply_action(starting_point: any) -> any:
        return action(action(starting_point))

    return apply_action


# the implementation, action being: add +1 to the passed number
def increment(number: int) -> int:
    return number + 1


repeat_plus_one = two(increment)
assert repeat_plus_one(starting_point=0) == 2
```

### 1.2. back to the lambdas with the two function

```python
two = lambda f: lambda x: f(f(x))
increment = lambda n: n + 1
repeated_increment = two(increment)
assert repeated_increment(0) == 2
```

### 1.3. example with tuple increment

```python
def p(t: tuple) -> tuple:
    return (t[0] + 1, t[0])


assert p((1, 0)) == (2, 1)
```
* redefine `three` with the nested lambdas

```python
three = lambda f: lambda x: f(f(f(x)))
```

* call previously defined `p` three times

```python
assert three(p)((0,0)) == (3,2)
assert three(p)((1,0)) == (4,3)
```

### 1.4. you can rewrite `lambda`-defined functions in python for explicity
* the above lambda implementation is the same as the named function implementation below
* the named function has to contain a return statement, the lamnda cannnot

```python
from typing import any


def three(fn: callable) -> callable:
    def inner(starting_point: any) -> any:
        return fn(fn(fn(starting_point)))

    return inner


result = three(lambda x: x + 1)(0)
assert result == 3
```

### 1.5. how do you implement zero?

```python
zero = lambda f: lambda x: x
result = zero(f=lambda n: n + 1)(x=999)
assert result == 999
```

* the implementation of `zero` is identital to the implementation of `false`

## 2. ARITHMETICS
### 2.1. successor
* what is the most minimal thing you need to do math?
* there are peano axioms for math
* the challenge is to implement successor
* given function `two`, return function `three`

```python
one = lambda f: lambda x: f(x)  # substitute for n below
two = lambda f: lambda x: f(f(x))  # substitute for n below
succ = lambda n: (lambda f: lambda x: f(n(f)(x)))  # n is church numeral
```
* the call is `succ(two)(increment)(2)` and it would return `5`
* let's write this out first

```python
from typing import any


def succ(church_numeral: callable) -> callable:
    def apply_church_numeral(action: callable) -> callable:
        def apply_action(starting_point: any) -> any:
            return action(church_numeral(action)(starting_point))

        return apply_action

    return apply_church_numeral


# church numeral
def two(action: callable) -> callable:
    def apply_action(starting_point: any) -> any:
        return action(action(starting_point))

    return apply_action


# action
def increment(n: int) -> int:
    return n + 1


# asserts
assert two(action=increment)(starting_point=0) == 2
assert succ(church_numeral=two)(action=increment)(starting_point=0) == 3
three = succ(two)
three_increments = three(increment)
result = three_increments(0)
assert result == 3
```


### 2.2. add

```python
from typing import any

increment = lambda n: n + 1
one = lambda f: lambda x: f(x)  # substitute for n below
two = lambda f: lambda x: f(f(x))  # substitute for n below
succ = lambda n: (lambda f: lambda x: f(n(f)(x)))  # n is church numeral


def add(addend1: callable) -> callable:
    def apply_add(addend2: callable) -> callable:
        return addend2(succ)(addend1)

    return apply_add


result = add(addend1=one)(addend2=two)(increment)(0)
assert result == 3
```

### 2.3. multiplication
* making m-times the n-times action 
* note that `mul` takes `f`, while `add` takes `f` from `succ`

```python
one = lambda f: lambda x: f(x)  # substitute for n below
two = lambda f: lambda x: f(f(x))  # substitute for n below
succ = lambda n: (lambda f: lambda x: f(n(f)(x)))  # n is church numeral
add = lambda x: lambda y: y(succ)(x)
mul = lambda x: lambda y: lambda f: y(x(f))
```
