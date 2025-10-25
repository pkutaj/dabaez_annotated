This is a final (and by far most challenging/grandious) installment of a multi_part series is to annotate David Baezley's [Lambda Calculus from the Ground Up](https://www.youtube.com/watch?v=pkCLMl0e_0k)
Part8 focuses on the implementation of a Y-combinator, building up on an implementation of a `FACTORIAL` function but removing
1. named variables
2. self-reference in recursion

It begins at [02h47m35s](https://youtu.be/pkCLMl0e_0k?si=q_Wp33yLeerg9Yx3&t=10055) and runs to the end of the great workshop.

## 1. IN PART7 WE'VE IMPLEMENTED `FACTORIAL` BUT WAIT - THERE WERE VARIABLES UP UNTIL NOW
> https://youtu.be/pkCLMl0e_0k?si=ejwDcR6fgNKs69J8&t=10055
* and lambda calculus does not allow global variables
* what we were doing with `FACT = ...` is not allowed
* because you are _storing_ the names somewhere
* this is a new programming problem when you start — finally — talking about **recursion**
* the idea of recursion is **referring to yourself**

### 1.1. let's start from a canonical definition of `factorial` as a recursive func

```python
fact = lambda n: 1 if n==1 else n*fact(n-1)
assert fact(3) == 6
```

### 1.2. challenge: rewrite that with no self-reference to `fact`
* how we can we **not have** that `fact` in the function definition?

### 1.3. approach 1: pass `fact` as an argument
* --> move the fact out into the outer part

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

### 1.4. solution: do the opposite you've been told about proper software - repeat yourself!
* use the idea above of using an outer lambda — but instead of a function call, rewrite the function definition
* which is he opposite of what you've been told about writing software: repeat yourself[1]
* just cut and paste the code and do it twice

```python
fact = (lambda f: lambda n:1 if n == 0 else n*f(n-1))\
       (lambda f: lambda n:1 if n == 0 else n*f(n-1))
fact(4)
```

* initially, this does blow up with a `TypeError`

```python
fact = (lambda f: lambda n:1 if n == 0 else n*f(n-1))\
       (lambda f: lambda n:1 if n == 0 else n*f(n-1))
                                             #^^^^^^^
fact(4) # FAIL
# TypeError: unsupported operand type(s) for *: 'int' and 'function'
```

* the thing that's wrong is that the "API" is not being followed
* the API is — pass function first, number second - the first attempt is passing just number
* --> you need an `f`, otherwise you are missing an argument in the broken call

### 1.5. solution that works

```python
fact = (lambda f: lambda n:1 if n == 0 else n*f(f)(n-1))\
       (lambda f: lambda n:1 if n == 0 else n*f(f)(n-1))
assert fact(4) == 24 # PASS 
```

#### 1.5.1. Initial Function Structure

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

#### 1.5.2. What `fact` Actually Is

```
fact = lambda n: 1 if n == 0 else n*f(n-1)
       └─ where f = (lambda f: lambda n: 1 if n == 0 else n*f(n-1))
```

#### 1.5.3. Call Stack Analysis: fact(4)

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

#### 1.5.4. The Error Occurs

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

#### 1.5.5. Visual Type Mismatch

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

#### 1.5.6. The Fix: f(f)(n-1)

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

## 2. FIXED POINTS
> https://youtu.be/pkCLMl0e_0k?si=G1Sa8v30bMT0T9FX&t=10765
* imagine pressing square root of a number repeatedly, say when you were bored at school
* pressing square root over and over again brings you eventually to one
> also [SICP > lecture 2A - this exact point!](https://youtu.be/eJeMOEiHv8c?si=G6AlSJvX9J5Dw4ep&t=1550)
* fixed point is a value that wen passed into a function you'll get the same value back
* passing `1` into square root function, you'll get the same thing back

```python
import math

999
for i in range(60):
    last_result = math.sqrt(last_result)
    print(last_result)
```

returns

```
31.606961258558215
5.62200687108778
2.371077154182837
1.53983023550742
1.2408989626506342
1.1139564455806314
1.0554413510852374
1.027346753090327
1.013581152690956
1.0067676756287698
1.0033781319267276
1.0016876418957796
...
1.0
1.0
1.0
1.0
```

* this is one of the most interesting things of all about lambda calculus - exploring fixed points
* this matters because the fix-point concept is the foundation of computational recursion

### 2.1. let's look at original factorial to start talking about fixed points'

```python
fact = lambda n: 1 if n == 0 else n*fact(n-1)
assert fact(4) == 24
```

### 2.2. we've removed `fact` from the function definition - inially just by wrapping the function into another lambda
* of course this only works if you have the original `fact` 👆 at hand

```python
fact = (lambda f: lambda n: 1 if n == 0 else n*f(n-1))(fact)
assert fact(4) == 24
```

### 2.3. take the middle part and put it into its own variable

```python 
R = lambda f: lambda n: 1 if n == 0 else n * f(n - 1)
fact = R(fact)
assert fact(4) == 24
```

* The 'API' or R is a two-stepp process, there is closure and currying again
* passing `fact` initially refers to the original `fact=lambda n: 1 if n==0 else n*fact(n-1)`
* the `R` closes over that original function and "remembers" it

### 2.4. sitestep: visualizing closure
* nothing other than function mechanism is allowed
* no packages, no objects, no numbers, no strings, no types - only functions; no operators
* can you compute anything if a single-argument function is the only thing in the universe?
* no control flow

![](Pasted%20image%2020250825064358.png)

### 2.5. Realize: in `fact=R(fact)`, `fact` must be a **fixed point of R**
* the whole idea of fixed is that you have a function where you pass something in --> you get identical thing back
* square root of one is one
* R(fact) gets you back to fact
* however, `fact` is not a value!
* we still do not know what `fact` is 

### 2.6. The idea of getting the to the fixed point
1. You **know the algorithm** or the solution beforehand
2. You cannot use self-reference because lambda calculus works only with anonymous functions
3. You need a function definition of the anonymous function that would be referenced 
4. You place the algorithm definition in the **template**
5. You need a fixed point - the function `f` such that `f = R(f)`

Previously, we found the fixed point by copy-pasting and following the "API" of the template. 
That does not always have to be the case - there is a specific higher order function that generates
the fixed point based on the template to have recursion. This is called **fixed point combinator**

### 2.7. LEAP: Suppose that there is a function Y that computes the fixed point of R
* Typical math trick — **supposed** there is such a thing that it exists, the function `Y`
* Another, yet even higher-order function
* You put your logic into the template `R`, but you don't know what the definition of self-reference would be
* You do not have the fixed point of R
* The `Y(R)` would be that -> generator of fixed point of R(whatever it is)
* Then `Y(R)` = `R(Y(R))` which would replace `fact`
* Then a little recursion trick

```python
Y(R) = R(Y(R)) # replace fact with supposed `Y`
Y(R) = (lambda f: R(f))(Y(R)) # recursion trick: pull it out into a variable and put it as an argument. 
Y(R) = (lambda f: R(f))(lambda f: R(f)) # repeat yourself trick -
# the above is incorrect
Y(R) = (lambda f: R(f(f))(lambda f: R(f)(f))
Y(R) = lambda
#...
```

* The derivation above creates `Y Combinator` invented by Haskell Curry
* In Python  it does not work because of infinite recursion error because it does eager evaluation
* The solution would be - ironically - solution to all python problems - you put a decorator on it

#### 2.7.1. Starting point: `fact` is a fixed point of a recursion template holding the program logic

```python
# original
fact = lambda n: 1 if n == 0 else n * fact(n-1)
# cannot have self-reference --> parametrize it
fact = (lambda f: lambda n: 1 if n == 0 else n*f(n-1))(fact)
# move the algorithm into `R` template
fact = R(fact) # this is not valid python; it's a mathematical contruct
```

* at this point, we know the logic is the problem
* we do not know the function definition of the referenced function
* - it cannot be identical as the problem definition as it needs to adopt its form
* ...to the parametrization of the function calls

#### 2.7.2. Wishful thinking: Suppose there is such a function that can generate a fixed point function
* from `fact = R(fact)` we have established that `R` is a fixed point type of function
* this is **not** a fixed point function that **iteratively converges** to a fixed point
    * ...like one used for finding statistical avefage, or cos(x) or square root
* this is about existence, the question 
    * IS NOT: what do you get if you iterate R repeatesly
    * INSTEAD: what function satisfies the constraint
* let's call this `Y`, where `Y(R)` computes a fixed point of `R`
* then if `R(fact)` is `fact` -> the fixed_point_getter `fact = Y(R)` and `Y(R) = R(Y(R))` !!!  

```python
# old
fact = R(fact)
# new: substitute a wished function to get a fixed point out of template
Y(R) = R(Y(R))
```

* we've abstracted `fact` away now by substitution/replacement

#### 2.7.3. Recursion trick on `fact` (extract self-reference into parameter)
* Pull a named function into a variable out of function definition / function call
* We've done it when we've removed the self-referring `fact` from the function definition

```python
# OLD: the 'recursion' trick on fact
fact = lamnda n: 1 if n == 0 else n*fact(n-1)
fact = (lambda f: lambda n: 1 if n == 0 else n*f(n-1))(fact)
# applied to
Y(R) = R(Y(R))
Y(R) = (lambda f: R(f))(Y(R))
```

#### 2.7.4. Repeat Yourself Trick
* Again, the same step done to `fact`, what you're told never to do: to repeat yourself

```
# What we WANT (but can't have):
fact(n-1)  # Self-reference by name

# What we GET instead:
f(f)(n-1)  # some_function(particular_function)(parameter_to_particular_function)
```

```python
# OLD: the `repeat yourself` step on fact
fact = (lambda f: lambda n: 1 if n == 0 else n*f(n-1))\
       (lambda f: lambda n: 1 if n == 0 else n*f(n-1))
# abstracted to the derivation of Y
Y(R) = (lambda f: R(f))\
       (lambda f: R(f))
```

#### 2.7.5. Fixing Repeat Yourself by fixing the API via adding a buffer f(f)
* the above is broken for both `fact` and `Y(R)`
* it is that there is a signature of the called function and you need to pass parameters it requires
* let's draw this for `fact`

![](Pasted%20image%2020250904071707.png)

* realize that in `fact` you are **calling** `f` that is being passed
* however in `Y(R)` derivation, `f` that is passed is an argument to the call of `R`
* to see the "API" we need to align with, we need to see the `R`

```python
# FIX: the `repeat yourself` step on fact
fact = (lambda f: lambda n: 1 if n == 0 else n*f(f)(n-1))\
       (lambda f: lambda n: 1 if n == 0 else n*f(f)(n-1))
# abstracted to the derivation of Y - but the above fix is not applied to `R` 
# R contains 'pure logic'
R = lambda f: lambda n: 1 if n == 0 else n*f(n-1)
Y(R) = (lambda f: R(f))\
       (lambda f: R(f))
```


```python
# substitution rule
Y(R) = (lambda f: R(f))(lambda f: R(f)) # pass inner_lambda into outer_lambda
       #^outer_lambda^ #^inner_lambda^ 
Y(R) = R(f=lambda f: R(f))
Y(R) = lambda n: 1 if n == 0 else n * (lambda f: R(f))(n-1)
                                      #^expects_funct|^passing_int
                                      #parameter_mismatch, value where function belongs!
```

* The fixed version is - note that it is not `R(f)(f)` - compare with previous bufferinG

```python
R = lambda f: lambda n: 1 if n == 0 else n*f(n-1)
Y(R) = (lambda f: R(f(f)))(lambda f: R(f(f)))
Y(R) = R(f(f)) # where f = (lambda f: R(f(f)))
```

* let's do the substituion rule

```py
Y(R) = R(
         argument
       )

       # where argument = (lambda f: R(f(f)))(lambda f: R(f(f)))
```

* the `argument` is a function calling another function

```python
Y(R) = R(
        function1(function2)
)
# where function1 = lambda f: R(f(f))
# where function2 = lambda f: R(f(f))
```

* to visualize the structure

```
Y(R) = R(
         ┌───────────────────┐   ┌───────────────────┐
         │ lambda f: R(f(f)) │ ( │ lambda f: R(f(f)) │ )
         └───────────────────┘   └───────────────────┘
         │                       │
         outer function          argument passed to it
       )
```

```py
Y(R) = R(
         lambda f: R(f(f))(lambda f: R(f(f)))
         #^outer_function  ^argument_passed 
         )
```
* but this would start an infinite recursion as the next substitution (simplified) would be

```python
Y(R) = R(
        lambda f: R(argument)
        # where argument = (lambda f: R(f(f)))(lambda f: R(f(f)))
)
```

* etc. etc. etc. - until you give it a number

#### 2.7.6. Recursion trick on `R` (extract self-reference into parameter)
* Before starting the infite recursion above 
* We buffered the call to ensure the 'API' works and were at
* The "looks like a formula" -> we can "pull out the R" 

```py
# PREVIOUS
Y(R) = (lambda f: R(f(f)))(lambda f: R(f(f)))
# apply 'recursion' trick on R -> extract R into a parameter, wrapped into another lambda
Y(R) = (lambda g:
            (lambda f: g(f(f)))(lambda f: g(f(f)))
        )(R)
```

#### 2.7.7. final: drop r (template) from both sides of the assignment/equation ~> y combinator

```python
# previous
y(r) = (lambda g: (lambda f: g(f(f)))(lambda f: g(f(f))))(r)
#^^^drop                                                 ^^^drop

# new
y = lambda g: (lambda f: g(f(f)))(lambda f: g(f(f)))
```

#### 2.7.8. notes on derived y combinator
* the way you use it
1. define recursive relationship/logic into a template
2. apply template to y combinator

```python
r = <logic_no_self_reference>
y = <const>
fact = y(r) # generate factorial recursive function
result = fact(<param>) # get result using the recursive function
```

* for example

```python
r = lambda f: lambda n: 1 if n == 0 else n * f(n-1)
y = lambda r: (lambda f: r(f(f)))(lambda f: r(f(f)))
fact = y(r)
result = fact(10)
```

