# Exercise 1.1

Below is a sequence of expressions. What is the result printed by
the interpreter in response to each expression? Assume that the sequence is to
be evaluated in the order in which it is presented. 

```lisp
10
(+ 5 3 4)
(- 9 1)
(/ 6 2)
(+ (* 2 4) (- 4 6))
(define a 3)
(define b (+ a 1))
(+ a b (* a b))
(= a b)
(if (and (> b a) (< b (* a b)))
    b
    a)
(cond ((= a 4) 6)
      ((= b 4) (+ 6 7 a))
      (else 25))
(+ 2 (if (> b a) b a))
(* (cond ((> a b) a)
         ((< a b) b)
         (else -1))
   (+ a 1))
```

## Answer

```lisp
10
; 10

(+ 5 3 4)
; 12

(- 9 1)
; 8

(/ 6 2)
; 3

(+ (* 2 4) (- 4 6))
; 6

(define a 3)
; a (and `a` is set to 3)

(define b (+ a 1))
; b (and `b` is set to 4)

(+ a b (* a b))
; 19

(= a b)
; #f

(if (and (> b a) (< b (* a b)))
    b
    a)
; 4

(cond ((= a 4) 6)
      ((= b 4) (+ 6 7 a))
      (else 25))
; 16

(+ 2 (if (> b a) b a))
; 6

(* (cond ((> a b) a)
         ((< a b) b)
         (else -1))
   (+ a 1))
; 16
```

# Exercise 1.2

Translate the following expression into prefix form:

$$ \frac {5 + 4 + (2 - (3 - (6 + \frac{4}{5})))}{3 (6 - 2) (2 - 7)} $$


## Answer

```lisp
(/ (+ 5
      4
      (- 2
         (- 3
            (+ 6
               (/ 4 5)))))
   (* 3 (- 6 2) (- 2 7)))
```

# Exercise 1.3

Define a procedure that takes three numbers as arguments and returns the sum of the squares of the two larger numbers. 

## Answer

```lisp
(define (square x) (* x x))

(define (sum-of-squares x y) (+ (square x) (square y)))

(define (sum-of-squares-of-two-largest x y z)
    (cond ((and (< x y) (< x z)) (sum-of-squares y z))
          ((and (< y x) (< y z)) (sum-of-squares x z))
          (else (sum-of-squares x y))))

(sum-of-squares-of-two-largest 2 3 4)
```

# Exercise 1.4

Observe that our model of evaluation allows for combinations whose operators are compound expressions. Use this observation to describe the behavior of the following procedure:

```lisp
(define (a-plus-abs-b a b)
    ((if (> b 0) + -) a b))
```

## Answer

The operator is an expression, so it is evaluated. If `b` is positive, the expression returns `+`, otherwise, it returns `-`.

The resulting operation is then applied to the `a` and `b` variables.


# Exercise 1.5

Ben Bitdiddle has invented a test to determine whether the interpreter he is faced with is using applicative-order evaluation or normal-order evaluation. He defines the following two procedures:

```lisp
(define (p) (p))

(define (test x y) 
  (if (= x 0) 
      0 
      y))
```

Then he evaluates the expression

```lisp
(test 0 (p))
```

What behavior will Ben observe with an interpreter that uses applicative-order evaluation? What behavior will he observe with an interpreter that uses normal-order evaluation? Explain your answer. (Assume that the evaluation rule for the special form if is the same whether the interpreter is using normal or applicative order: The predicate expression is evaluated first, and the result determines whether to evaluate the consequent or the alternative expression.) 

## Answer

The `p` procedure is an infinite recursive loop. If it is evaluated, the program will hang (and eventually crash).

Following applicative order, the operands are evaluated first. So when `(p)` is evaluated, the program hangs.

On the other hand, following normal order, the following expansion happens:

```lisp
(test 0 (p))

; `test` operation body is substituted
(if (= 0 0) 
      0 
      (p)))

; Predicate is evaluated
(if #t
    0
    (p))

; `if` returns the consequent
0
```

This happens because `if` is a special form which only evaluates the appropriate operand depending on the predicate. If `if` followed the default rule of evaluation, all operands would be evaluated, so `(p)` would execute, and then the program would hang just like it did with applicative order.

# Exercise 1.6

Alyssa P. Hacker doesn’t see why if needs to be provided as a special form. “Why can’t I just define it as an ordinary procedure in terms of cond?” she asks. Alyssa’s friend Eva Lu Ator claims this can indeed be done, and she defines a new version of if:

```lisp
(define (new-if predicate 
                then-clause 
                else-clause)
  (cond (predicate then-clause)
        (else else-clause)))
```

Eva demonstrates the program for Alyssa:

```lisp
(new-if (= 2 3) 0 5)
; 5

(new-if (= 1 1) 0 5)
; 0
```

Delighted, Alyssa uses new-if to rewrite the square-root program:

```lisp
(define (sqrt-iter guess x)
  (new-if (good-enough? guess x)
          guess
          (sqrt-iter (improve guess x) x)))
```

What happens when Alyssa attempts to use this to compute square roots? Explain. 


## Answer

The proceedure gets stuck in an infinite loop. Because `new-if` is not a special form, and because the interpreter is following applicative order for evaluation, its operands are evaluated before being passed on to the procedure.

Because of this, both `then-clause` and `else-clause` are executed, no matter what the value of the predicate.

This makes no difference with the examples Eva gives at first, since those clauses simply return a primitive value. But in the `sqrt-iter` procedure, this causes the recursion to always be triggered, leading to an infinite loop (and eventual crash).


# Exercise 1.7

The `good-enough?` test used in computing square roots will not be very effective for finding the square roots of very small numbers. Also, in real computers, arithmetic operations are almost always performed with limited precision. This makes our test inadequate for very large numbers. Explain these statements, with examples showing how the test fails for small and large numbers. An alternative strategy for implementing good-enough? is to watch how guess changes from one iteration to the next and to stop when the change is a very small fraction of the guess. Design a square-root procedure that uses this kind of end test. Does this work better for small and large numbers? 

## Answer

### Problems with large numbers

Real numbers are (usually) implemented in computers with [floating-point arithmetic](https://en.wikipedia.org/wiki/Floating-point_arithmetic). That kind of arithmetic has many quirks ([some examples](https://jvns.ca/blog/2023/01/13/examples-of-floating-point-problems/)), one of which being that they have limited precision, which changes depending on how large the number is.

For large numbers, the precision can get low to the point that the difference between one representable number and the next is larger than the defined $ 0.001 $ threshold. That means that the operation would never reach the stop condition, and would go on forever.

In `mit-scheme`, real numbers are implemented with double-precision (64 bit) floating-point numbers. In those, the smallest number for which the gap between it and the next representable number gets larger than $ 0.001 $ is $ 2 ^{43} $ (see [this](./fp-gaps.md) for how that was calculated).

Sure enough, `(sqrt (- (expt 2.0 43) 1))` returns correctly, while `(sqrt (expt 2.0 43))` hangs indefinitey.


### Problems with small numbers

For very small numbers, the problem is not FP numbers, but just math itself. If the number we want to find the square root for is smaller than $ 0.001 $, that means that we will reach the stop condition while the guess is still very innacurate.

As an example, the square root for $ 0.00001 $ comes out as $ 0.0313 $, when it actually is $ 0.0031 $, because $ 0.0313^2 = 0.00097 $. Although this is much larger than the number we want, it is within the defined static margin of error.


### A better `good-enough?`

```lisp
(define (good-enough? guess x)
  (< (abs (- (square guess) x)) (* 0.001 x)))
```

This works much better for small numbers. For the previos example of $ 0.00001 $, the result is pretty much exact.

For larger numbers, though, there are pros and cons. As a pro, the procedure always returns now, instead of hanging above a certain value.

The problem now is that $ 0.001 * \text{a large number} $ is still a very large number. So the final result might be very far from correct value.


# Exercise 1.8

Newton’s method for cube roots is based on the fact that if $ y $ is an approximation to the cube root of $ x $, then a better approximation is given by the value

$$ \frac{x / y^2 + 2y}{3} $$

Use this formula to implement a cube-root procedure analogous to the square-root procedure. (In 1.3.4 we will see how to implement Newton’s method in general as an abstraction of these square-root and cube-root procedures.) 

## Answer

All that was needed was to change the `improve` definition, and to make the `good-enough?` predicate compare the cube of the guess to the original value, rather than the square.

```lisp
; Main call
(define (cbrt x) (cbrt-iter 1.0 x))

; Recursion iteration
(define (cbrt-iter guess x)
  (if (good-enough? guess x)
      guess
      (cbrt-iter (improve guess x) x)))

; Predicate for stop condition
(define (good-enough? guess x)
  (< (abs (- (cube guess) x)) (* 0.001 x)))

; Get a better guess
(define (improve guess x)
  (/ (+ (/ x (square guess))
        (* 2 guess))
     3))

; Helper procedures
(define (square x) (* x x))
(define (cube x) (* x x x))
```
