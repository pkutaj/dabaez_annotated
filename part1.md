---
id: part1
aliases: []
tags: []
---

The aim of this multi_part series is to annotate David Baezley's [Lambda Calculus from the Ground Up](https://www.youtube.com/watch?v=pkCLMl0e_0k)
This part is about the first 35 mins of the lecture. Use the link above to start watching. Use Python Repl to follow. 

## 1. INTRO
* only single-argument functions are allowed
* nothing other than function mechanism is allowed
* no packages, no objects, no numbers, no strings, no types - only functions; no operators
* can you compute anything if a single-argument function is the only thing in the universe?
* no control flow

## 2. SWITCH

```python
def left_switch(left: Any) -> Callable:
    def get_right(right: Any) -> Any:
        return left

    return get_right


assert left_switch("a")("b") == "a"


def right_switch(left: Any) -> Callable:
    def get_right(right: Any) -> Any:
        return right

    return get_right


assert right_switch("a")("b") == "b"
```


### 2.1. explain currying via example: rewriting two-argument func into single-argument ones

```python
def add(x:int, y:int) -> int:
    return x + y


assert add(1, 1) == 2


def add(x: int) -> callable:
    def f(y: int) -> int:
        return x + y
    return f

# CURRYING
assert add(1)(1) == 2
```

## 3. TRUE/FALSE
![](pasted%20image%2020250408193642.png)

* can we represent `TRUE` and `false` with this scheme?
* similarly to switch, or similarly to how computers work:
* truth is 1 - it is electrical device.
* false is 0
--> you can make logical machine with that
* it is either electricity or ground on a bit

```python
def true(x):
    return lambda y: x  # selector chooses the front


assert true(1)(0) == 1


def false(x):
    return lambda y: y  # selector choosing the back


assert false(1)(0) == 0
```

* it is amazing that you don't need the name of the function
* so that the function through returns a function that takes one parameter
* and the cold with another call
* and return the first value that you give it
* here, truth is behavior it is not correspondence - these are **opposite** behaviors
* there is nothing **concrete** to hang on; whatever the x is

### 3.1. anonymous functions as products have this neat freeing effect
* so you don't need to think about the name for the function
* the main function of the whole name

## 4. BOOLEANS: not, or, and

single boolean transformation

### 4.1. NOT
##### 4.1.0.1. hello world
* realize the difference between calling() and object **test** 
* there is **no currying** here


```python
def not(x: callable) -> callable: # accepts only true/false functions
    return x(false)(true) # only 2 possible scenarios
    # scenario 1: true(false)(true) returns false
    # scenario 2: false(false)(true) returnes true

not(true)
# out[76]: <function __main__.false(x) -> <built-in function callable>>

not(false)
# out[77]: <function __main__.true(x: str) -> <built-in function callable>>

assert not(true)(1)(0) == 0 # ok
assert not(false)(1)(0) == 1 # ok
```

* so why is that
* because not is not returning a value
* your pass a function
* i need calls the function with two parameters that are also functions

### 4.2. OR
* you call twice again, you combine bool functions/pickers --> there has to be a lambda for the second arg

```python
def or(picker1):
    return lambda picker2: picker1(picker1)(picker2)
```

* realize that you know what you want to achieve in advance, the logic is what is represented already!

```python
or(true)(false)        # --> function true
or(true)(true)         # --> function true
or(false)(true)        # --> function true
or(false)(false)       # --> function false
or(false)(true)(1)(0)  # 1
or(true)(true)(1)(0)   # 1
or(true)(false)(1)(0)  # 1
or(false)(false)(1)(0) # 0
```

```python
def or(x): 
    return lambda y: x(x)(y)

```
### 4.3. AND

 | x     | y     | result | selector |
 | ----- | ----- | ------ | -------- |
 | true  | true  | true   | 2        |
 | true  | false | false  | 2        |
 | false | true  | false  | 1        |
 | false | false | false  | 1        |

```python
def and(x):
    return lambda y: x(y)(x)

and(true)(true)   # --> true
and(true)(false)  # --> false
# let's subsitute
    # true(false)(true)
    # false
and(false)(true)  # --> false
and(false)(false) # --> false
```

