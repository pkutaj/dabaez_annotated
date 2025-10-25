---
id: part3
aliases: []
tags: []
---

The aim of this multi_part series is to annotate David Baezley's [Lambda Calculus from the Ground Up](https://www.youtube.com/watch?v=pkCLMl0e_0k)
This part is a digression from lambda calculus into a related/applied field of utilizing a concept similar to peano numbers to a getter of a nested data.
It covers 10 mins of the workshop starting at ~01h:08min:07s –> https://youtu.be/pkCLMl0e_0k?si=biIoF1uLDbpe3zmB&t=4087 and going up to the break at ~01h:18min:42sec


## 1. DIGRESSION
* note below that `getc` is also again, again, again on the outer dictionary

```python
from typing import any

two = lambda f: lambda x: f(f(x))

data = {"a": {"b": {"c": 42}}}


def getc(d: dict) -> any:
    return d["a"]["b"]["c"]


assert getc(data) == 42
```

* could i apply a church numeral there? 
* you would have to call the key multiple times
* so the function with me give me the key
* presupposing you only have a one key
* but how can i get a key out of the dictionary in python
* if i don't know the name of the key
* this is the function with data already having a starting point


### 1.1. apply many if statements so that program does not crash
```python
from typing import any

data = {"a": {"b": {"c": 42}}}

def getc(d: dict) -> any:
    d = d.get("a")
    if d is not none:
        d = d.get("b")
    if d is not none:
        d = d.get("c")
    return d


assert getc(d=data) == 42
```

### 1.2. add `perhaps` higher-order function (hof) either running passed func or returning none
* the pattern is having a hof, and that hof accepts data and function
* within the hof, the function is eventually called, with data as argument

```python
from typing import union, any

data = {"a": {"b": {"c": 42}}}


def perhaps(d: dict, func: callable) -> union[any, none]:
    if d is not none:
        return func(d)
    else:
        return none


assert perhaps(d=data, func=lambda d: d.get("a")) == {"b": {"c": 42}}  # pass
assert perhaps(d={}, func=lambda d: d.get("a")) == none  # pass
```

### 1.2.1. add **chaining** to the `perhaps` getting nearer to recursion
* chaining means that you pass a function call of the same function `perhaps` into
    the `d` argument
* this is recursive pattern, and you already have a base case in `perhaps`

```python
from typing import union, any

data = {"a": {"b": {"c": 42}}}


def perhaps(d: dict, func: callable) -> union[any, none]:
    if d is not none:
        return func(d)
    else:
        return none


assert perhaps(
    d=perhaps(d=data, func=lambda d: d.get("a")), func=lambda d: d.get("b")
) == {"c": 42}

assert (
    perhaps(
        d=perhaps(
            d=perhaps(d=data, func=lambda d: d.get("a")), func=lambda d: d.get("b")
        ),
        func=lambda d: d.get("c"),
    )
    == 42
)
assert result == 42
```


### 1.2.2. introduce the `class perhaps` to fix the chaining of `perhaps` function

```python
from typing import union, any
data = {
    'a': {
        'b': {
            'c': 42
            }
        }
}

class perhaps:
  def __init__(self,value):
    self.value = value

  def __rshift__(self,other):
    if self.value is not none:
      return perhaps(other(self.value))
    else:
      return self

perhaps(data) >> (lambda d:d.get('a'))
```

### 1.2.3. perhaps this is a monad
* associated with "turning a crank" on lots of function compositions
* monad is like a song: perhaps, perhaps, perhaps
