# Notes

## 1.1 The Elements of Programming

>  Every powerful language has three mechanisms for accomplishing this:
>
> - primitive expressions, which represent the simplest entities the language is concerned with,
> - means of combination, by which compound elements are built from simpler ones, and
> - means of abstraction, by which compound elements can be named and manipulated as units.


### 1.1.1 Expressions

Prefix notation: leftmost element is the **operator**, all other elements are **operands**.

```lisp
(+ 53 12) ; Evaluates to 65
(/ 10 2)  ; Evaluates to 5
```


Can take arbitrary number of arguments:

```lisp
(+ 1 2 3 4 5) ; Evaluates to 15
```


Expressions can be nested:

```lisp
(+ (* 3 (+ (* 2 4) (+ 3 5))) (+ (- 10 7) 6)) ; Evaluates to 57
```


Pretty printing makes it easier to understand:

```lisp
(+ (* 3
      (+ (* 2 4)
         (+ 3 5)))
   (+ (- 10 7)
      6))
```


### 1.1.2 Naming and the Environment

Use `define` to declare a variable and set a value to it.

```lisp
(define size 5)
size            ; Evaluates to 5
(* size 2)      ; Evaluates to 10
```

> It should be clear that the possibility of associating values with symbols and later retrieving them means that the interpreter must maintain some sort of memory that keeps track of the name-object pairs. This memory is called the environment (more precisely the global environment, since we will see later that a computation may involve a number of different environments).


### 1.1.3 Evaluating Combinations

Emphasis mine:

> To **evaluate** a combination, do the following:
>
> 1. **Evaluate** the subexpressions of the combination.
> 2. Apply the procedure that is the value of the leftmost subexpression (the operator) to the arguments that are the values of the other subexpressions (the operands). 
>
> [...]
>
> Thus, the evaluation rule is *recursive* in nature; that is, it includes, as one of its steps, the need to invoke the rule itself.


The vast majority of expressions follow this rule, but some do not. Those that don't are called **special forms**.

> For instance, evaluating (define x 3) does not apply define to two arguments, one of which is the value of the symbol x and the other of which is 3, since the purpose of the define is precisely to associate x with a value. (That is, (define x 3) is not a combination.)

Each special form has its own rule for how to be evaluated.

Lisp has simple syntax (or as some put it, "no syntax"), because the way it's written closely resembles the [AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree). Other languages have syntax which are quite diferent from their ASTs.


### 1.1.4 Compound Procedures

Before, we saw how to abstract primitive values by given them names with `define`.

**Procedure definitions** can in turn give names to compound operations.

```lisp
(define (square x) (* x x))
(square 2)          ; Evaluates to 4
(square (+ 5 2))    ; Evaluates to 49
(square (square 3)) ; Evaluates to 81
```

The general form of a procedure definition is:

```lisp
(define (<name> <formal parameters>) <body>)
```


### 1.1.5 The Substitution Model for Procedure Application

> We can assume that the mechanism for applying primitive procedures to arguments is built into the interpreter. For compound procedures, the application process is as follows:
>
> To apply a compound procedure to arguments, evaluate the body of the procedure with each formal parameter replaced by the corresponding argument.

The following is the sequence of substitutions that would be taken to evaluate the first expression:

```lisp
(+ (square 6) (square 10))
(+ (* 6 6) (* 10 10))
(+ 36 100)
136
```

This is called the **substitution model** for procedure application.

It is just that: a model. It does not describe how the interpreter actually works, and has limitations - it breaks down when dealing with mutable data, for example. Other (more complex/accurate) models will be presented when they're needed, but for now, it works, and it's very simple.


#### Applicative order versus normal order

For the following procedure:

```lisp
(define (sum-of-squares x y) (+ (square x) (square y))
```

The following would be applying the substitution model like we did before, which is called **applicative order**. In this model, operator and operands are evaluated, and the results are applied to the procedure:

```lisp
(sum-of-squares (+ 5 1) (* 5 2))
(sum-of-squares 6 10)            ; Operands evaluated
(+ (square 6) (square 10))       ; `sum-of-squares` procedure body substituted
(+ (* 6 6) (* 10 10))            ; `square` procedure body substituted
(+ 36 100)                       ; Operands evaluated 
136                              ; Final result
```

While the following would be using **normal order**, in which operands are not evaluated until their values are needed:

```lisp
(sum-of-squares (+ 5 1) (* 5 2))
(+ (square (+ 5 1)) (square (* 5 2)))       ; `sum-of-squares` substituted
(+ (* (+ 5 1) (+ 5 1)) (* (* 5 2) (* 5 2))) ; `square` substituted
                                            ; (Only primitive values and 
                                            ; operations left)
(+ (* 6 6) (* 10 10))                       ; Operands evaluated
(+ 36 100)                                  ; Operands evaluated
136                                         ; Final result
```

For procedures that can be modeled using the substitution model, both of those methods always produce the same results.

Lisp uses applicative order because:

- It's more efficient (note how `(+ 5 1)` was calculated twice in the normal order example)
- Normal order becomes complicated to deal with when leaving the realm substitution models

Normal order does have advantage in some applications, such as stream processing.


### 1.1.6 Conditional Expressions and Predicates

There are two constants, `#t` and `#f`, which represent True and False boolean values. The names `true` and `false` are set to these values as well.

`#t` is included mostly for the sake of convenience and simmetry: for the purpose of boolean logic, every value besides `#f` is considered truthy.

There are two special forms for expressing conditionals: `cond` and `if`.

```lisp
(cond (<predicate 1> <consequent 1>)
      (<predicate 2> <consequent 2>
      ...
      (<predicate n> <consequent n>))

(define (abs x)
  (cond ((> x 0) x)
        ((= x 0) x)
        ((< x 0) (- x))))
```

`if` is a special case of `cond`, for when there are only two possibilities:

```lisp
(if <predicate> <consequent> <alternative>)

(define (abs x) (if (< x 0) (- x) x))
```

The following are the logical composition operations:

```lisp
(and <e1> <e2> ... <en>)
(or <e1> <e2> ... <en>)
(not <e>)
```

`and` and `or` are special forms: not all subexpressions are evaluated because of [short-circuiting](https://en.wikipedia.org/wiki/Short-circuit_evaluation)


### 1.1.7 Example: Square Roots by Newton’s Method

| Mathematics | Computer Science |
|:-----------:|:----------------:|
| Functions   | Procedures       |
| What is     | How to           |
| Declarative | Imperative       |

As a concrete example of the difference between Mathematics and Computer Science, the book uses square root.

Function definition for the square root:

$$ \sqrt{x} = \text{the } y \text{ such that } y \ge 0 \text{ and } y^2 = x $$

That's what the square root *is*, but it tells us nothing about how to actually calculate this $ y $.

For that, we need a procedure. The following is an implementation of Newton's Method, using recursion:

```lisp
; Main call
(define (sqrt x) (sqrt-iter 1.0 x))

; Recursion iteration
(define (sqrt-iter guess x)
  (if (good-enough? guess x)
      guess
      (sqrt-iter (improve guess x) x)))

; Predicate for stop condition
(define (good-enough? guess x)
  (< (abs (- (square guess) x)) 0.001))

; Get a better guess
(define (improve guess x) (average guess (/ x guess)))

; Helper procedures
(define (average x y) (/ (+ x y) 2))
(define (square x) (* x x))
```


# 1.1.8 Procedures as Black-Box Abstractions

The beauty of procedures is that they can be used without knowing how they're actually implemented. 

Substituting an implementation for a completely different one should make no difference for the user of that procedure: all that matters is that the results are correct:

```lisp
; Using
(define (square x) (* x x))

; should be indistinguishable to using
(define (square x) 
  (exp (* (log x) 2)))
```

## Local names

The names of a procedure's formal parameters are local to that procedure.

This has two consequences. The first is that the following definitions are exactly equivalent:

```lisp
(define (square x) (* x x))
(define (square y) (* y y))
```

The second is that different procedures can reuse the same name without causing conflicts:

```lisp
(define (good-enough? guess x)
  (< (abs (- (square guess) x)) 0.001))

(define (square x) (* x x))
```

Both those procedures use the `x` name. But you can see that `good-enough?` passes its `guess` argument to `square`.

This means that in a call like `(good-enough? 2 5)`, in the body of `good-enough?` the name `x` will have the value `5`, while in the body of `square` it will be `2`.

In other words, it doesn't matter what the names are given for the formal parameters of a procedure. Those names are independent from any other names defined outside of the procedure, and are called *bound variables*. Renaming a bound variable has no effect on the functioning of the procedure.

Variables which are not bound are said to be *free variables*. The procedure does depend on those names being defined somewhere outside of itself, and changing them does potentialy change the way the procedure works (e.g., renaming `abs` to `cos` in the previous example).

The set of expresions fow which a variable is bound is called the *scope* of that name. For scope for formal parameters is the body of the procedure they're defined in.


## Internal definitions and block structure

In the previous example with `sqrt`, we defined many helper procedures to break down the task into smaller steps.

A user of that procedure does not care about those inner steps, though: the only procedure that matters to them is `sqrt`. More over, they may want to define their own procedures with names we used, such as `good-enough?`, but as it is, by using our `sqrt`, all those names are already taken.

To fix that, we can "hide" those steps by making those definitions internal to `sqrt`:

```lisp
(define (sqrt x)
  (define (good-enough? guess x)
    (< (abs (- (square guess) x)) 0.001))

  (define (improve guess x)
    (average guess (/ x guess)))

  (define (sqrt-iter guess x)
    (if (good-enough? guess x)
        guess
        (sqrt-iter (improve guess x) x)))
```

This nesting of defintions is called *block structure*.

Going further, we may notice that `x` is defined in the `sqrt` defintion, and is thus defined (i.e., in scope) in its body. Since all substeps simply use that value as is, there is no need to redefine that name and pass it explicitly:

```lisp
(define (sqrt x)
  (define (good-enough? guess)
    (< (abs (- (square guess) x)) 0.001))

  (define (improve guess)
    (average guess (/ x guess)))

  (define (sqrt-iter guess)
    (if (good-enough? guess)
        guess
        (sqrt-iter (improve guess))))
```
