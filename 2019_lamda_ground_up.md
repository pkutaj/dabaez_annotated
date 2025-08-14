---
id: 2019_lamda_ground_up
aliases: []
tags: []
---
## TODOs
- [ ] add video timestamps

## INTRO
* only single-argument functions are allowed
* nothing other than function mechanism is allowed
* no packages, no objects, no numbers, no strings, no types - only functions; no operators
* can you compute anything if a single-argument function is the only thing in the universe?
* no control flow

## 1. SWITCH

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


### 1.1. rewriting two-argument func into single-arg with the technique known as currying

```python
def add(x, y):
    return x + y


assert add(1, 1) == 2


def add(x: int) -> callable:
    def f(y: int) -> int:
        return x + y

    return f


assert add(1)(1) == 2
```

## 2. TRUE/FALSE
![](pasted%20image%2020250408193642.png)

* can we represent `TRUE` and `false` with this scheme?
* similarly to switch, or similarly to how computers work:
* truth is 1 - it is electrical device.
* false is 0
--> you can make logical machine with that
* it is either electricity or ground on a bit

```python
def true(x):
    return lambda y: x  # vyberatko vrati predok


assert true(1)(0) == 1


def false(x):
    return lambda y: y  # vyberatko vrati zadok


assert false(1)(0) == 0
```

* it is amazing that you don't need the name of the function
* so that the function through returns a function that takes one parameter
* and the cold with another call
* and return the first value that you give it
* there were teachers saying that lambda should be a key on the keyboard
* here, truth is behavior it is not correspondence - these are **opposite** behaviors
* there is nothing **concrete** to hang on; whatever the x is

### 2.1. anonymous functions as products have this neat freeing effect
* so you don't need to think about the name for the function
* the main function of the whole name

## 3. BOOLEANS: not, or, and

single boolean transformation

### NOT
##### 3.1.1.1. hello world
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

### OR
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
###  AND

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


## 4. NUMBERS
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

### 4.1. fail > replacing lambdas with named functions stressing the abstractness of number
* let's put come concreteness to the abstraction so that abstraction is clearer
* below is my initial thinking that i sincorre t

```python
from typing import Any


def two(action: callable) -> callable:
    def init(starting_point: int) -> Any:
        return action(action(starting_point))

    return init


assert two(action=lambda n: n + 1)(starting_point=0) == 2
```

### 4.2. pass > the `init` name is incorrect - the point of `init` is to apply the `action` from the first param

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

### 4.3. back to the lambdas with the two function

```python
two = lambda f: lambda x: f(f(x))
increment = lambda n: n + 1
repeated_increment = two(increment)
assert repeated_increment(0) == 2
```

### 4.4. pass > example with tuple increment

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

### 4.5. you can rewrite `lambda`-defined functions in python for explicity
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

### 4.6. fail > possibly a one-liner? no, that is a syntax error as `def` is not allowed,
* it is a compoud statement

```python
def three(fn:callable) -> callable: def inner(starting_point:any): return fn(fn(fn(starting_point))); return inner
#   cell in[19], line 1
#     def three(fn:callable) -> callable: def inner(starting_point:any): return fn(fn(fn(starting_point))); return inner
#                                         ^
# syntaxerror: invalid syntax
```

* therefore, for condensed expressions like this there are lambdas,
* but you need to know how to read

### 4.7. pass > how do you implement zero?

```python
zero = lambda f: lambda x: x
result = zero(f=lambda n: n + 1)(x=999)
assert result == 999
```

* the implementation of `zero` is identital to the implementation of `false`

## 5. ARITHMETICS
### 5.1. successor
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


### 5.2. add

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

### 5.3. multiplication
* making m-times the n-times action 
* note that `mul` takes `f`, while `add` takes `f` from `succ`

```python
one = lambda f: lambda x: f(x)  # substitute for n below
two = lambda f: lambda x: f(f(x))  # substitute for n below
succ = lambda n: (lambda f: lambda x: f(n(f)(x)))  # n is church numeral
add = lambda x: lambda y: y(succ)(x)
mul = lambda x: lambda y: lambda f: y(x(f))
```

## 6. DIGRESSION
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


### 6.1. aim: recurse up to the value with the `values()`
#### 6.1.1. pass > test the method to return everything after the key: --> `list`
```python
data.values()
# [ins] in [6]: data.values()
# out[6]: dict_values([{'b': {'c': 42}}])
```

#### 6.1.2. fail > assign to an iterator object and iterate through `next`: there is only 1 iteration

```python
data_iterator=iter(data.values())
data_iterator
# [ins] in [8]: data_iterator
# out[8]: <dict_valueiterator at 0x106fb2fc0>
```
* use the `next` function

```python
next(data_iterator)
# [ins] in [9]: next(data_iterator)
# out[9]: {'b': {'c': 42}}
#
# [ins] in [10]: next(data_iterator)
# ---------------------------------------------------------------------------
# stopiteration                             traceback (most recent call last)
# cell in[10], line 1
# ----> 1 next(data_iterator)
#
# stopiteration: 
```

### 6.2. back: you need to make every nested key a part of iteration - start with dabaez
#### 6.2.1. pass > apply many if statements so that program does not crash
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

#### 6.2.2. add `perhaps` higher-order function either running passed func or returning none
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

#### 6.2.3. pass > add **chaining** to the `perhaps` getting nearer to recursion
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


#### 6.2.4. introduce the `class perhaps` to fix the chaining of `perhaps` function

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

#### 6.2.5. perhaps this is a monad
* associated with "turning a crank" on lots of function compositions
* monad is like a song: perhaps, perhaps, perhaps

## 7. SYMBOLS
### 7.1. it is possible to minimize function definition

```python
# most explicit and longest
def and(x):
  def f(y):
    return x(y)(x)
  return f

# enter nested lambda
def and(x):
  return lambda y: x(y)(x)

# pure lambdas
and = lambda x: lambda y: x(y)(x)

# nested lambdas can be combined
and = lambda xy: x(y)(x)
```
```python
def TRUE(x):
    return lambda y: x  # vyberatko vrati predok


assert TRUE(1)(0) == 1


def FALSE(x):
    return lambda y: y  # vyberatko vrati zadok


assert FALSE(1)(0) == 0


# most explicit and longest
def AND(x):
    def f(y):
        return x(y)(x)

    return f


assert AND(TRUE)(TRUE)(1)(1) == 1  # TRUE -> vyberatko vrati predok
assert AND(TRUE)(FALSE)(1)(0) == 0  # FALSE -> vyberatko vrati zadok


# enter nested lambda
def AND(x):
    return lambda y: x(y)(x)


assert AND(TRUE)(TRUE)(1)(1) == 1  # TRUE -> vyberatko vrati predok
assert AND(TRUE)(FALSE)(1)(0) == 0  # FALSE -> vyberatko vrati zadok

# pure lambdas
AND = lambda x: lambda y: x(y)(x)

assert AND(TRUE)(TRUE)(1)(1) == 1  # TRUE -> vyberatko vrati predok
assert AND(TRUE)(FALSE)(1)(0) == 0  # FALSE -> vyberatko vrati zadok

# nested lambdas can be combined
AND = lambda xy: x(y)(x)
assert AND(TRUE)(TRUE)(1)(1) == 1  # TRUE -> vyberatko vrati predok
assert AND(TRUE)(FALSE)(1)(0) == 0  # FALSE -> vyberatko vrati zadok

assert AND(TRUE)(TRUE)(1)(1) == 1  # TRUE -> vyberatko vrati predok
assert AND(TRUE)(FALSE)(1)(0) == 0  # FALSE -> vyberatko vrati zadok
```


## 8. CONVERSIONS
### 8.1. why is this called calculus: calculus is about conversions
* in most known definitions, calculus is about "rate of change"
* here, it has nothing to do with what - so why is it called calculus?
* lambda - symbology. way of writing a function.
    * variables next to each other, etc.
* calculus - there are bunch of **conversions** taking place in lambda expressions
    there are things you can do with functions
* you can substitute thins in the symboloy
### 8.2. Rule1: Alpha conversion - renamingjgoes
* You can rename an argument,
* if var `z` is used, it is reserved
* caveat: can't introduce a name clash

### 8.3. Rule2: Argument substitution goes
* you can substitute arguments
* take what's in parenthesis and substitute
* this is the essence of function call
    local `x` and `y` could be called `a` and `b` and the result is the same

```python
x=3
y=4

def f(x,y):
  return 2*x + y

assert f(x,y) == 10
```

### 8.4. Rule3: You can make a function

```python
x=3
x=(lambda a:a)(3)
assert x == 3
```

### 8.5. Why are we talking about rules of lambda?
* in the early 20th century there was a lots of discussion about the nature of Math
* can all of mathematics be derived from symbolic manipulation?
* if you had proper set of axioms - can you derive all of math
* main figure: David Hilberg (1862-1943)
* no creativity at all. with right set of axioms, could you just turn the crank and 
* solve everything
#### 8.5.1. Entscheidunsproblem
* is there an algorithm that takes
    * statement of first-order logic as an input
    * return boolean (yes/no) according to whether the statement is *universally valid*
    * i.e. satisfying all of the axioms
* in other words - is there an algorithm to decide whether a given statement is
    * provable from the axioms, using the rules of logic

#### 8.5.2. Responses: Goedel, Church, Turing; 1930s
1. Goedel - incompleteness theorem
2. Church - lambda kalkul - no way you can get axiomatic system
3. Turing - turing machines (equivalent to lambda kalkul)
.
.
.
McCarthy (Lisp) -> Djikstra -> SICP -> Closure -> UncleBob -> Primagean

#### 8.5.3. Lisp
We're using lambda notation 
Lisp paper is talking about AI - is there any other mention of lambda kalkul
we're borrowing notation
lambda is a convenient way to express functions
Lisp has a connection

## 9. LISTS
### 9.1. 1957 0 IMB 704(1957)
* in lambda kalkul world, is there a way to build datastructure
* how can you do list in lambda kalkul?
* you can build datastructures similarly like you build switch -> boolean expression -> boolean logic
-> numbers
* the `cons`, `car` and `cdr` names are from the names of hardware parts from IBM704
* you build a data structure entirely from function definition

### 9.2. cons, car, cdr

```python
from typing import Any
def cons(a: Any,b: Any) -> callable:
  def select(m: int) -> Any:
    if m == 0: return a
    if m == 1: return b
  return select
assert cons(2,3)(0) == 2
data = cons(2,3)
assert data(0) == 2
assert data(1) == 3
car = lambda d: d(0)
cdr = lambda d: d(1)
data = cons(111,222)
assert car(data) == 111
assert cdr(data) == 222
```

* the question seems to be so what
* and the answer seems to be that you have a structure based on behavior
* which is similar to having true/false purely on behavior and convention/consensus and building a logical system from that axiom
* I know Professor Sussman going into the implementation of dysfunction and saying that it "out of thin air" 
* there is nothing in it, just action, all of which you can build to things
* this is probably something along these lines

### 9.3. back to the beginnng and the switch
* the pair requires three Parameters
* One parameter is the first value, the second parameter is the second value, and the third parameter is a function
* Therefore, you know, the lambda calculus function has two nested functions

```python
from typing import Any

def cons(a: Any) -> callable:  #make_tuple
  def apply_select(b: Any) -> callable: #make_select
     def select(selector:callable) -> Any: #select
       return selector(a)(b)
     return select
  return apply_select

def car(x: Any) -> callable:
  def inner(y: Any) -> Any:
    return x
  return inner

def cdr(x: Any) -> callable:
  def inner(y: Any) -> Any:
    return y
  return inner

p = cons(2)(3)
assert car(1)(2) == 1
assert cons(2)(3)(car) == 2
assert p(car) == 2
assert p(cdr) == 3

```
* this is a mind blowing idea - just from a function you can build a data structure
* sort of out of nothingness, in SICP they said out of thin air

### 9.4. As opposed to what? Traditional code-to-metal mapped DSs.
* As opposed to implementing code that would map the data used to hardware
* remember cs50 and the basic explanation of an array: you place data in contiguous registers in memory
* the memory is a physical box, you have "cranes" that put things in there and take out "things" ouf of there
* here, you don't care about hardware at all - machine is irrelevant. it's the mind that matters

### 9.5. Enter closure: The second item of a pair can be a list
See /Users/mr_paul/kb/sicp/05_01-Closure.md for closure definition. 
This is the foundation
It works and it is extensible
The second item of first pair is the rest of the list

```python
from typing import Any
CONS = lambda a: lambda b: lambda fn: fn(a)(b)
CAR = lambda a: lambda b: a
CDR = lambda a: lambda b: b

pair = CONS(1)(2)
assert pair(CAR) == 1
assert pair(CDR) == 2
TRUE = lambda x: lambda y: x
FALSE = lambda x: lambda y: y
CAR = lambda fn: fn(TRUE)
pair = CONS(1)(2)
assert CAR(pair) == 1
```
* to be continued at
https://g.co/gemini/share/984aeec14303
* Let's use the substitution
* This was so important for functional programming
* It was known as a stepper and the rule was called substitution rule

```python
from typing import Any
CONS:callable = lambda a: lambda b: lambda selector: selector(a)(b)
pair:callable = CONS(1)(2) # note it is 2 argument, wairing for 
CAR: callable = lambda pair: pair(selector=lambda a: lambda b: a)
CDR: callable = lambda pair: pair(selector=lambda a: lambda b: b)
assert CAR(pair) == 1
assert CDR(pair) == 2
```

* Rewritten in long-hand

```python
from typing import Any
def CONS(car: Any) -> Any:
  def _(cdr: Any) -> callable:
    def select_one(selector: callable) -> Any:
      return selector(car)(cdr)
    return select_one 
  return _

def CAR(CONS:callable) -> Any:
  def select_first(item_1: Any) -> callable: # can be λa:λb:a
    def _(item_2: Any) -> callable:
      return item_1
    return _
  return CONS(select_first) # can be function-def-in-function-call - see above

pair1 = CONS(1)(2)
assert CAR(pair1) == 1

def CDR(CONS:callable) -> Any:
  def select_second(item_1:Any) -> callable: # can be λa:λb:b
    def _(item_2:Any) -> Any:
      return item_2
    return _
  return CONS(select_second) # function-name in function-as-param

pair2 = CONS(3)(4)
assert CDR(pair2) == 4
```

### 9.6. there are three ways you can use functions as parameter
Within the parameter you can pass these three variations:

1. Function name
2. Function definition
3. Function call

The previous section has function **name** as a parameter
Below is functional **definition** as a paramater


```python
from typing import Any
def CONS(car:Any) -> callable: 
  def _(cdr:Any) -> callable:
    def pair(selector:callable) -> Any:
      return selector(car)(cdr)
    return pair
  return _
pair = CONS(0)(1)
def CAR(pair:callable): return pair(selector=lambda a:lambda b: a) # funcion_def in function-as-param
assert CAR(pair) == 0
def CDR(pair:callable): return pair(selector=lambda a: lambda b: b) # function_def in function-as-param
assert CDR(pair) == 1
```

* In the language of SICP cons function is a constructor
* The other functions are selectors
* The combination of these two + closure is what allows building complex system

## 10. SUBTRACTIONS
### 10.1. WIP > Let's rewrite basic pair functions CONS and CAR and church numeral + SUCC: understand T=...
* this is just to repeat and rewrite in lambda functions
```python
# pair
CONS = lambda a: lambda b: lambda selector: selector(a)(b)
# selector functions
CAR = lambda pair: pair(selector=lambda a: lambda b: a)
CDR = lambda pair: pair(selector=lambda a: lambda b: b)
pair1 = CONS(1)(2)
assert CAR(pair=pair1) == 1
assert pair1(selector=lambda a:lambda b: a) == 1
assert CAR(pair=CONS(1)(2)) == 1
# reimplement def increment(pair: tuple) -> tuple: return (pair[0]+1, pair[0])
SUCC = lambda church_numeral: lambda fn: lambda init_value: fn((church_numeral)(fn)(init_value))
    # church_numeral is church numeral
    # fn is any function you want to apply church_numeral times
    # init_value is the init input for the function
ZERO = lambda fn: lambda init_value: init_value
ONE = SUCC(ZERO)
TWO = SUCC(ONE)
THREE = SUCC(TWO)
#THREE = lambda fn: lambda init_value: fn(fn(fn(init_value)))
plusone = lambda n: n+1
assert plusone(0) == 1
assert THREE(fn=plusone)(init_value=1) == 4
assert SUCC(church_numeral=THREE)(fn=plusone)(init_value=2) == 6
FOUR = SUCC(church_numeral=THREE)
assert FOUR(fn=plusone)(init_value=1) == 5
# this is NEW
FIVE = SUCC(church_numeral=FOUR)
assert FIVE(fn=plusone)(init_value=1) == 6
assert CAR(pair1) == 1
```

### 10.2. how can we subtract?
* how do you get from one number to previous number?
* so if `THREE` is the following ~> how would you get `TWO`?
* the idea could be
    - 1. start with (0,0) where [0] is current, [1] is previous value
    - 2. ratchet your way up to three

```
(1,0) ONE
(2,1) TWO
(3,2) THREE
```

* in python without lambda calculus this could be something like using tuples

```python
def increment(pair: tuple) -> tuple: return (pair[0]+1, pair[0])

a = (0,0)

assert THREE(fn=increment)(init_value = (0,0)) == (3,2)
assert increment(a) == (1,0)
```

this is the lambda version from dabaez ~> with the previous number using `CDR`

```python
# given pair of 
# create a new pair which
make_incremented_CONS=lambda pair: CONS(SUCC(CAR(pair)))((CAR)(pair))
a=THREE(fn=make_incremented_CONS)(CONS(ZERO)(ZERO))
assert CAR(a)(plusone)(0) == 3
assert CDR(a)(plusone)(0) == 2 # get a previous number
```

### 10.3. let's rewrite into proper function - make_incremented_CONS longhand

```python
# given a pair created by CONS funcion
# create another pair containing only the first item or the pair where
# the first item is an succeessor of the first item
# the second item is the original value of the first item
def make_incremented_CONS(CONS_PAIR:callable) -> callable: # creator function
    return CONS( 
                SUCC(CAR(CONS_PAIR)), # first item is success of the first item
                CAR(CONS_PAIR) # second item is the first item
               )
# define a church numeral with
# `make_incremented_CONS` as a function to repeat
# a CONS_pair as an init value (not a regular church numeral)
a=THREE(fn=make_incremented_CONS)(init_value=CONS(ZERO)(ZERO)) # returns CONS(THREE)(TWO)
b=CAR(a) # expects THREE as church numeral
c=CDR(a) # expecte TWO   as church numeral
assert b(fn=plusone)(init_value=0) == 3
assert c(fn=plusone)(init_value=0) == 2
```


### 10.4. Use `make_incremented_CONS` to make a PREDeccessor as opposite to SUCCessor

```python
from typing import Any
def get_previous_value(church_numeral:callable) -> Any: # selector function
  return CDR(church_numeral(make_incremented_CONS)(CONS(ZERO)(ZERO)))

d=get_previous_value(church_numeral=THREE)
assert d(fn=plusone)(init_value=0) == 2 # PASS
```

### 10.5. Call Stack Visualized

```
get_previous_value(church_numeral=THREE)
│
└── CDR(THREE(fn=make_incremented_CONS)(init_value=CONS(ZERO)(ZERO)))
    │
    ├── THREE(fn=make_incremented_CONS)(init_value=CONS(ZERO)(ZERO))
    │   │
    │   │   THREE = λfn.λinit_value.fn(fn(fn(init_value)))
    │   │   fn = make_incremented_CONS
    │   │   init_value = CONS(ZERO)(ZERO)  // Initial pair: (0,0)
    │   │
    │   ├── Call 1: make_incremented_CONS(CONS(ZERO)(ZERO))
    │   │   │
    │   │   │   Input:  CONS(ZERO)(ZERO)     // pair: (0,0)
    │   │   │   CAR:    ZERO                 // first element: 0
    │   │   │   SUCC(CAR): ONE               // successor of first: 1
    │   │   │   
    │   │   └── Output: CONS(ONE)(ZERO)      // new pair: (1,0)
    │   │
    │   ├── Call 2: make_incremented_CONS(CONS(ONE)(ZERO))
    │   │   │
    │   │   │   Input:  CONS(ONE)(ZERO)      // pair: (1,0)
    │   │   │   CAR:    ONE                  // first element: 1
    │   │   │   SUCC(CAR): TWO               // successor of first: 2
    │   │   │   
    │   │   └── Output: CONS(TWO)(ONE)       // new pair: (2,1)
    │   │
    │   ├── Call 3: make_incremented_CONS(CONS(TWO)(ONE))
    │   │   │
    │   │   │   Input:  CONS(TWO)(ONE)       // pair: (2,1)
    │   │   │   CAR:    TWO                  // first element: 2
    │   │   │   SUCC(CAR): THREE             // successor of first: 3
    │   │   │   
    │   │   └── Output: CONS(THREE)(TWO)     // final pair: (3,2)
    │   │
    │   └── Final result: CONS(THREE)(TWO)
    │
    └── CDR(CONS(THREE)(TWO))
        │
        │   CDR = λpair.pair(λa.λb.b)
        │   pair = CONS(THREE)(TWO)
        │   selector = λa.λb.b  // selects second element
        │
        └── Result: TWO  // The predecessor of THREE!

═══════════════════════════════════════════════════════════════════

Summary

Initial State:     (0, 0)
                    ↓ make_incremented_CONS
After Call 1:      (1, 0)  ← SUCC(0)=1, previous=0
                    ↓ make_incremented_CONS  
After Call 2:      (2, 1)  ← SUCC(1)=2, previous=1
                    ↓ make_incremented_CONS
After Call 3:      (3, 2)  ← SUCC(2)=3, previous=2
                    ↓ CDR (select second element)
Final Result:       TWO    ← The predecessor of THREE!

═══════════════════════════════════════════════════════════════════

Function Breakdown:
- CONS(a)(b) creates a pair containing a and b
- CAR(pair) extracts the first element  
- CDR(pair) extracts the second element
- make_incremented_CONS(pair) → CONS(SUCC(first))(first)
- Church numeral applies a function n times to an initial value
```

### 10.6. Combining Church Numerals

```python
SIXTEEN=TWO(fn=FOUR) # -> lambda fn: lambda init_value: FOUR(FOUR(init_value)) -> SIXTEEN
assert SIXTEEN(plusone)(0) == 16 # PASS
```

#### 10.6.1. applying substitition rule
```
# Goal: Evaluate TWO(FOUR)(plusone)(0)

# Definitions:
# TWO       = lambda fn: lambda init_value: fn(fn(init_value))
# FOUR      = lambda fn: lambda init_value: fn(fn(fn(fn(init_value))))
# plusone   = lambda n: n + 1

# --------------------------------------------------------------------------------

# Initial Expression:
# TWO(FOUR)(plusone)(0)

# Step 1: Apply 'TWO' to 'FOUR'
#         The definition of TWO is `lambda fn: lambda init_value: fn(fn(init_value))`
#         We are calling TWO with 'FOUR' as the argument for 'fn'.

#         Expression before substitution:
#         (lambda fn: lambda init_value: fn(fn(init_value)))(FOUR)

#         Substitute 'fn' with 'FOUR' in the body `lambda init_value: fn(fn(init_value))`.
#         The outer `lambda fn:` is consumed by the application.

# Expression after Step 1:
# (lambda init_value: FOUR(FOUR(init_value)))(plusone)(0)
# (This lambda expression is effectively our 'SIXTEEN' Church numeral)

# Step 2: Apply the result (lambda init_value: FOUR(FOUR(init_value))) to 'plusone'
#         The function is `lambda init_value: FOUR(FOUR(init_value))`
#         We are calling it with 'plusone' as the argument for 'init_value'.

#         Expression before substitution:
#         (lambda init_value: FOUR(FOUR(init_value)))(plusone)

#         Substitute 'init_value' with 'plusone' in the body `FOUR(FOUR(init_value))`.
#         The `lambda init_value:` is consumed by the application.

# Expression after Step 2:
# (FOUR(FOUR(plusone)))(0)

# Step 3: Evaluate the innermost 'FOUR(plusone)'
#         The definition of FOUR is `lambda fn: lambda init_value: fn(fn(fn(fn(init_value))))`
#         We are calling FOUR with 'plusone' as the argument for 'fn'.

#         Expression before substitution:
#         FOUR(plusone)  -->  (lambda fn: lambda init_value: fn(fn(fn(fn(init_value)))))(plusone)

#         Substitute 'fn' with 'plusone' in the body `lambda init_value: fn(fn(fn(fn(init_value))))`.
#         The outer `lambda fn:` is consumed.

# Result of Step 3:
# lambda init_value: plusone(plusone(plusone(plusone(init_value))))
# Which simplifies to: lambda init_value: init_value + 4
# Let's call this intermediate function 'add_four_fn'

# Expression after Step 3:
# (FOUR(add_four_fn))(0)

# Step 4: Evaluate the outer 'FOUR(add_four_fn)'
#         The definition of FOUR is `lambda fn: lambda init_value: fn(fn(fn(fn(init_value))))`
#         We are calling FOUR with 'add_four_fn' as the argument for 'fn'.

#         Expression before substitution:
#         FOUR(add_four_fn) --> (lambda fn: lambda init_value: fn(fn(fn(fn(init_value)))))(add_four_fn)

#         Substitute 'fn' with 'add_four_fn' in the body `lambda init_value: fn(fn(fn(fn(init_value))))`.
#         The outer `lambda fn:` is consumed.

# Result of Step 4:
# lambda init_value: add_four_fn(add_four_fn(add_four_fn(add_four_fn(init_value))))

#         Let's trace what this means for 'init_value':
#         add_four_fn(init_value)                       = init_value + 4
#         add_four_fn(add_four_fn(init_value))          = (init_value + 4) + 4 = init_value + 8
#         add_four_fn(add_four_fn(add_four_fn(init_value))) = (init_value + 8) + 4 = init_value + 12
#         add_four_fn(add_four_fn(add_four_fn(add_four_fn(init_value)))) = (init_value + 12) + 4 = init_value + 16

#         So, this simplifies to: lambda init_value: init_value + 16
#         Let's call this intermediate function 'add_sixteen_fn'

# Expression after Step 4:
# add_sixteen_fn(0)

# Step 5: Apply the final function 'add_sixteen_fn' to '0'
#         The function is `lambda init_value: init_value + 16`
#         We are calling it with '0' as the argument for 'init_value'.

#         Expression before substitution:
#         (lambda init_value: init_value + 16)(0)

#         Substitute 'init_value' with '0' in the body `init_value + 16`.
#         The outer `lambda init_value:` is consumed.

# Final Result:
# 0 + 16
# 16
```

### 10.7. add church_numeral combination with `PRED`

```python
EIGHTYONE = FOUR(THREE)
e = get_previous_value(EIGHTYONE)
assert e(plusone)(0) == 80
```

### 10.8. PASS > with `fn` being plustwo, the previous value is `160`: previous value of a fn call
* forget about numbers, this is about **output**

```python
assert FOUR(THREE)(lambda n: n + 2)(0) == 162
assert get_previous_value(FOUR(THREE))(lambda n: n + 2)(0) == 160
```

## 11. SUBTRACTION
### 11.1. how to define custom function in the most easiest way: use `typing.Callable`
* for the matter of clarity, I would like to simply have a type hint of a custom type
* I don't need an implementation of fuctom type, but if it is a church numeral function,
* I would like to haave a function definintion of something like

### 11.2. let's build up dependencies

```python
from typing import Callable, Any

church_numeral = Callable[..., Any]
#datastructure - pairs
CONS=lambda a: lambda b: lambda s: s(a)(b)
CAR=lambda p: p(lambda a: lambda b: a)
CDR=lambda p: p(lambda a: lambda b: b)

p=CONS(1)(2)
assert CAR(p) == 1
assert CDR(p) == 2

#peano arithmetics: ZERO + SUCCESSOR
ZERO = lambda fn: lambda init: init
SUCC = lambda n: lambda fn: lambda init: fn(n(fn)(init))

ONE = SUCC(ZERO)
TWO = SUCC(ONE)
PLUSONE = lambda n: n+1
assert ONE(PLUSONE)(0) == 1

#peano arithmetics: incremented CONS DS + PREDECESSOR function
ICONS = lambda p: CONS(SUCC(CAR(p)))(CAR(p))
PRED = lambda x: CDR(x(ICONS)(CONS(ZERO)(ZERO)))

assert TWO(PLUSONE)(0) == 2
assert PRED(TWO)(PLUSONE)(0) == 1
```

### 11.3. and now, the new thing - using `PRED` for subtraction
* to substract x-y, you need to do `y` applications of `PRED` to `x`
* to subtract 5-2, you need to do two applications of pred to five
* five applications of `PRED` to `two`
* subtraction is not comutative, the order of operations matter greatly: insight!

```python
def SUB(x: church_numeral) -> church_numeral:
    def _(y: church_numeral) -> church_numeral:
        return y(PRED)(x)

    return _

assert SUB(TWO)(ONE)(PLUSONE)(0) == 1
THREE = SUCC(TWO)
FOUR = SUCC(THREE)
FOUR(PLUSONE)(0) == 4
assert (SUB(FOUR)(TWO))(PLUSONE)(0) == 2
```

## 12. TEST FOR ZERO
* if you can test number for ZERO it allows you to start checking things
* why?
* if you can start doing tests, you can start thinking about control flow
* like if statement - test, and select one of two branches, essential for computability
* uncle bob reference to paper how can programing be reduced
### 12.1. test for zero: the build_up

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
### 12.2. test for zero: the addition
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

### THE ASSEMBLY
* up until here, we have created an "assembly code" of functions

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
* but having that - you can have more interesting things like writing recurive functions

### FACTORIAL WITH RECURSION
* first, let's start with factorial in a python way

#### Usual Python implementation of factorial

```python
def fact(n):
    if n == 0:
        return 1
    else:
        return n * fact(n - 1)


assert fact(3) == 6
```

#### The new thing
* take the number
* do `ISZERO` check and return church boolean
* church booleans work like an if statement 

```python
def FACT(n: church_numeral) -> church_numeral:
  ISZERO(n) \
    (ONE) \
    (MUL(n)(FACT(PRED(n))))
```


#### the build_up: ISZERO, we need church_numerals + church_booleans

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

#### the build_up: `MUL`: 

```python
MUL = lambda x: lambda y: lambda f: y(x(f))
TWO = SUCC(ONE)
THREE = SUCC(TWO)
assert MUL(THREE)(TWO)(lambda n: n+1)(0) == 6
```

#### the build_up: `PRED`

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


#### finished with the build_up ~> let's compose a first recursive function

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

#### explaining why the above is not working in python
does substition work in python same as it would in math/lisp


```python
def f(1+2): return 3*x+1
assert f(3) == 10
```

* so how does that work exactly
* it does the argument first and then it calls the function
* python evaluates arguments first: first `1+2` then pass into function
*

#### hypothesis from claude: this is python's eager evaluation
The problem seems to be the order of execution

```python
FACT = lambda n: ISZERO(n)(ONE)(MUL(n)(FACT(PRED(n))))
```

Should be: 
1. eval `ISZERO` **only** - once you get result which is `TRUE`/`FALSE`
2. eval `TRUE(ONE)(MUL(n)(FACT(PRED(n))))`

Is: everything is evaluated **eagerly**, then first function is evaluated. 

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

### dabez explanation
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

### how can we turn that off the eager evaluation: move expression into a 0 arg function
* let's say we want to turn of `1/a`
* --> shove it in to 0 arg function, delay the calculation in some later point in time
* this is exactly how "function as parameters" work - they are called only when needed

```python
a = 0
f = lambda: 1 / a  # nothing happens
f()  # explodes upon call
# ZeroDivisionError: division by zero
```

### apply the "trick" to the `choose` function

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

### is this common for other languages: LLM to check which language does not do that
Dabaez says almost every programming language does it the same way

### how to fix: implement lazy evaluation to the `FACT` function
introduce extra function call in there
it is not part of lambda calculus - it is hack to not to both branches of `if`
this concept of "when functions evaluate" is a big focus in functional programming
we are facing it here

### apply the lambda wrapper to all of the elements of `FACT` what may break the functionality

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

## WAIT - THERE WERE VARIABLES UP UNTIL NOW
> https://youtu.be/pkCLMl0e_0k?si=ejwDcR6fgNKs69J8&t=10055
* lambda calculus does not allow global variables
* what we were doing with `FACT = ...` is not allowed
* because you are _storing_ the names somewhere
* this is a new programming problem when you start — finally — talking about **recursion**
* the idea of recursion is **referring to yourself**

### given a canonical recursive function: factorial

```python
fact = lambda n: 1 if n==1 else n*fact(n-1)
assert fact(3) == 6
```

### challenge: rewrite that with no self-reference to `fact`
* how we can we **not have** that `fact` in the function definition?

### approach 1: pass `fact` as an argument
* move the fact out into the outer part

```python
fact = lambda n: 1 if n==1 else n*fact(n-1)                  # OLD
fact = (lambda f: lambda n: 1 if n == 0 else n*f(n-1))(fact) # NEW — depends on OLD! 
```

which is pretty much the same as having a function returning the exact value of a parameter

```python
y = 10
x = (lambda x: x)(10)
assert x == y
```

as noted, the `NEW` `fact` could not exist without the previous declaration of `fact`

```python
del fact # unset a var
fact = (lambda f: lambda n: 1 if n == 0 else n*f(n-1))(fact) # NEW
# NameError: name 'fact' is not defined
```

-> the assignment semantics in python first evaluate the left side of `=` and `fact` does not exist at that point


### solution: do the opposite you've been told about proper software - repeat yourself!
* use the idea above of using an outer lambda — but instead of a function call, rewrite the function definition
* which is he opposite of what you've been told about writing software: repeat yourself[1]
* just cut and paste the code and do it twice

```python
fact = (lambda f: lambda n:1 if n == 0 else n*f(n-1))\
       (lambda f: lambda n:1 if n == 0 else n*f(n-1))
fact(4)
```

* initially, this does blow up with

```
TypeError: unsupported operand type(s) for *: 'int' and 'function'
```

```python
fact = (lambda f: lambda n:1 if n == 0 else n*f(n-1))\
       (lambda f: lambda n:1 if n == 0 else n*f(n-1))
                                             #^^^^^^^
fact(4)
```

* the thing that's wrong is that the "API" is not being followed
* the API is — pass function first, number second - the first attempt is passing just number
* --> you need an `f`, otherwise you are missing an argument in the broken call
* the fix 

```python
fact = (lambda f: lambda n:1 if n == 0 else n*f(f)(n-1))\
       (lambda f: lambda n:1 if n == 0 else n*f(f)(n-1))
assert fact(4) == 24
```

#### Initial Function Structure

```
fact = (lambda f: lambda n: 1 if n == 0 else n*f(n-1))(lambda f: lambda n: 1 if n == 0 else n*f(n-1))
       └─────────────── outer ───────────────────────┘└────────────── f-param ──────────────────────┘
```

Let's break this down:

```
outer_function = lambda f: lambda n: 1 if n == 0 else n*f(n-1)
f-param        = lambda f: lambda n: 1 if n == 0 else n*f(n-1)

fact = outer_function(f-param)
```

#### What `fact` Actually Is

```
fact = lambda n: 1 if n == 0 else n*f(n-1)
       └─ where f = (lambda f: lambda n: 1 if n == 0 else n*f(n-1))
```

#### Call Stack Analysis: fact(4)

**Stack Frame 1: fact(4)**

```
┌─────────────────────────────────────────┐
│ FRAME 1: fact(4)                        │
│                                         │
│ n = 4                                   │
│ f = (lambda f: lambda n: ...)           │
│                                         │
│ Executing: 1 if n == 0 else n*f(n-1)    │
│ Since n=4 ≠ 0, compute: 4 * f(3)        │
│                                         │
│ Calling: f(3)  ← PROBLEM STARTS HERE    │
└─────────────────────────────────────────┘
```

**What f Expects vs What It Gets**

```
f signature: lambda f: lambda n: ...
                    ↑         ↑
                    param1    param2

f(3) call:   f(3)
             ↑
             only 1 argument!

Expected:    f(some_function, 3)
Got:         f(3)
```

#### The Error Occurs

```
┌─────────────────────────────────────────┐
│ FRAME 2: f(3) - BROKEN CALL             │
│                                         │
│ f = 3  ← f parameter gets value 3!      │
│ n = ??? ← n parameter gets nothing!     │
│                                         │
│ Executing: 1 if n == 0 else n*f(n-1)    │
│                                         │
│ Since n is undefined, Python uses       │
│ default behavior and continues...       │
│                                         │
│ When it tries n*f(n-1):                 │
│ n is a function object                  │
│ f is 3 (integer)                        │
│ f(n-1) tries to call 3(function-1)      │
│ → TypeError!                            │
└─────────────────────────────────────────┘
```

#### Visual Type Mismatch

```
Expected computation:  4 * (recursive_result)
                       ↑     ↑
                      int    int

Actual computation:    4 * f(n-1)
                       ↑   ↑
                      int  3(function_object - 1)
                           ↑
                           trying to call integer 3 as function
```

#### The Fix: f(f)(n-1)

```
f(f)(n-1) breakdown:
├─ f(f)     ← first application: gives f its own function
│  └─ returns: lambda n: 1 if n == 0 else n*f(f)(n-1)
└─ (n-1)    ← second application: gives the result the number

Call stack with fix:
┌─────────────────────────────────────────┐
│ FRAME 1: fact(4)                        │
│ n = 4                                   │
│ Computing: 4 * f(f)(3)                  │
│ ├─ f(f) returns a function              │
│ └─ that function called with (3)        │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│ FRAME 2: recursive call with n=3       │
│ n = 3                                   │
│ Computing: 3 * f(f)(2)                 │
└─────────────────────────────────────────┘
                    │
                    ▼
            (continues recursively...)
```
[1] https://wiki.c2.com/?DontRepeatYourself
