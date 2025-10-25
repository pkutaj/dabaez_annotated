---
id: part5
aliases: []
tags: []
---

# PART5
> https://medium.com/@pavolkutaj/the-annotated-lambda-calculus-from-the-ground-up-part-5-67a167c21808
The aim of this multi_part series is to annotate David Baezley's [Lambda Calculus from the Ground Up](https://www.youtube.com/watch?v=pkCLMl0e_0k).
Part5 follows up [part2](https://medium.com/@pavolkutaj/part2-bf44d0fc4f7b) and implementing of basic arithmetic operations with lambda calculus. 
Part5 focuses on subtraction. Starting at [02h12m20s](https://youtu.be/pkCLMl0e_0k?si=l4LArQa6NwpEAY3t&t=7940) it covers only ~10minutes. 
For me, this is entering a more dense/complicated territory, so there's lots of longhand and perhaps too repetitive motions. 


## 1. SUBTRACTIONS
### 1.1. Let's rewrite basic pair functions CONS and CAR and church numeral + SUCC
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

### 1.2. how can we subtract
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

### 1.3. let's rewrite into proper function - make_incremented_CONS longhand

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


### 1.4. Use `make_incremented_CONS` to make a PREDeccessor as opposite to SUCCessor

```python
from typing import Any
def get_previous_value(church_numeral:callable) -> Any: # selector function
  return CDR(church_numeral(make_incremented_CONS)(CONS(ZERO)(ZERO)))

d=get_previous_value(church_numeral=THREE)
assert d(fn=plusone)(init_value=0) == 2 # PASS
```

### 1.5. Call Stack Visualized (LLM_assisted)

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

### 1.6. Combining Church Numerals

```python
SIXTEEN=TWO(fn=FOUR) # -> lambda fn: lambda init_value: FOUR(FOUR(init_value)) -> SIXTEEN
assert SIXTEEN(plusone)(0) == 16 # PASS
```

#### 1.6.1. applying substitition rule (LLM_assisted)

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

### 1.7. add church_numeral combination with `PRED`

```python
EIGHTYONE = FOUR(THREE)
e = get_previous_value(EIGHTYONE)
assert e(plusone)(0) == 80
```

### 1.8. PASS > with `fn` being plustwo, the previous value is `160`: previous value of a fn call
* forget about numbers, this is about **output**

```python
assert FOUR(THREE)(lambda n: n + 2)(0) == 162
assert get_previous_value(FOUR(THREE))(lambda n: n + 2)(0) == 160
```

### 1.9. how to define custom function in the most easiest way: use `typing.Callable`
* for the matter of clarity, I would like to simply have a type hint of a custom type
* I don't need an implementation of fuctom type, but if it is a church numeral function,
* I would like to haave a function definintion of something like

### 1.10. let's build up dependencies

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

### 1.11. and now, the new thing - using `PRED` for subtraction
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

