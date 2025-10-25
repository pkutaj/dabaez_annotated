The aim of this multi_part series is to annotate David Baezley's [Lambda Calculus from the Ground Up](https://www.youtube.com/watch?v=pkCLMl0e_0k)
This is the opening of the second part of the workshop

## 1. SYMBOLS
### 1.1. it is possible to minimize function definition

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
    return lambda y: x  # selector returns a first item


assert TRUE(1)(0) == 1


def FALSE(x):
    return lambda y: y  # selector returns a second item


assert FALSE(1)(0) == 0


# most explicit and longest
def AND(x):
    def f(y):
        return x(y)(x)

    return f


assert AND(TRUE)(TRUE)(1)(1) == 1  # TRUE -> selector returns a first item
assert AND(TRUE)(FALSE)(1)(0) == 0  # FALSE -> selector returns a second item


# enter nested lambda
def AND(x):
    return lambda y: x(y)(x)


assert AND(TRUE)(TRUE)(1)(1) == 1  # TRUE -> selector returns a first item
assert AND(TRUE)(FALSE)(1)(0) == 0  # FALSE -> selector returns a second item

# pure lambdas
AND = lambda x: lambda y: x(y)(x)

assert AND(TRUE)(TRUE)(1)(1) == 1  # TRUE -> selector returns a first item
assert AND(TRUE)(FALSE)(1)(0) == 0  # FALSE -> selector returns a second item

# nested lambdas can be combined
AND = lambda xy: x(y)(x)
assert AND(TRUE)(TRUE)(1)(1) == 1  # TRUE -> selector returns a first item
assert AND(TRUE)(FALSE)(1)(0) == 0  # FALSE -> selector returns a second item

assert AND(TRUE)(TRUE)(1)(1) == 1  # TRUE -> selector returns a first item
assert AND(TRUE)(FALSE)(1)(0) == 0  # FALSE -> selector returns a second item
```


## 2. CONVERSIONS
### 2.1. why is this called calculus: calculus is about conversions
* in most known definitions, calculus is about "rate of change"
* here, it has nothing to do with what - so why is it called calculus?
* lambda - symbology. way of writing a function.
    * variables next to each other, etc.
* calculus - there are bunch of **conversions** taking place in lambda expressions
    there are things you can do with functions
* you can substitute thins in the symboloy

### 2.2. Rule1: Alpha conversion - renamingjgoes
* You can rename an argument,
* if var `z` is used, it is reserved
* caveat: can't introduce a name clash

### 2.3. Rule2: Argument substitution goes
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

### 2.4. Rule3: You can make a function

```python
x=3
x=(lambda a:a)(3)
assert x == 3
```

### 2.5. Why are we talking about rules of lambda?
* in the early 20th century there was a lots of discussion about the nature of Math
* can all of mathematics be derived from symbolic manipulation?
* if you had proper set of axioms - can you derive all of math
* main figure: David Hilberg (1862-1943)
* no creativity at all. with right set of axioms, could you just turn the crank and 
* solve everything

#### 2.5.1. Entscheidunsproblem
* is there an algorithm that takes
    * statement of first-order logic as an input
    * return boolean (yes/no) according to whether the statement is *universally valid*
    * i.e. satisfying all of the axioms
* in other words - is there an algorithm to decide whether a given statement is
    * provable from the axioms, using the rules of logic

#### 2.5.2. Responses: Goedel, Church, Turing; 1930s
1. Goedel - incompleteness theorem
2. Church - lambda kalkul - no way you can get axiomatic system
3. Turing - turing machines (equivalent to lambda kalkul)

```
McCarthy (Lisp) -> Djikstra -> SICP -> Closure -> UncleBob -> Primagean
```

#### 2.5.3. Lisp
We're using lambda notation 
Lisp paper is talking about AI - is there any other mention of lambda kalkul
we're borrowing notation
lambda is a convenient way to express functions
Lisp has a connection

## 3. LISTS
### 3.1. 1957 0 IMB 704(1957)
* in lambda kalkul world, is there a way to build datastructure
* how can you do list in lambda kalkul?
* you can build datastructures similarly like you build switch -> boolean expression -> boolean logic
-> numbers
* the `cons`, `car` and `cdr` names are from the names of hardware parts from IBM704
* you build a data structure entirely from function definition

### 3.2. cons, car, cdr

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

### 3.3. back to the beginnng and the switch
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

### 3.4. As opposed to what? Traditional code-to-metal mapped DSs.
* As opposed to implementing code that would map the data used to hardware
* remember cs50 and the basic explanation of an array: you place data in contiguous registers in memory
* the memory is a physical box, you have "cranes" that put things in there and take out "things" ouf of there
* here, you don't care about hardware at all - machine is irrelevant. it's the mind that matters

### 3.5. Enter closure: The second item of a pair can be a list
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

### 3.6. there are three ways you can use functions as parameter
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
