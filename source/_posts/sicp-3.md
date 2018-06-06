---
title: sicp-3
date: 2016-11-25 09:54:24
categories: programming
tags: course
mathjax: true
---



#### Three.Modularity, Objects, and State(objects, streams)

##### **3.1Assignment and Local State**

> The trouble here is that substitution is based ultimately on the no-
> tion that the symbols in our language are essentially names for values.
> But as soon as we introduce set! and the idea that the value of a vari-
> able can change, a variable can no longer be simply a name. Now a
> variable somehow refers to a place where a value can be stored, and the
> value stored at this place can change. In Section 3.2 we will see how
> environments play this role of “place” in our computational model.
>
> 简而言之(lambda (x) (set! x (+ x 1))) 没法将 值 x为5代入（替代模型失效）

```lisp
(define new-withdraw (let ((balance 100))
(lambda (amount)
(if (>= balance amount)
(begin (set! balance (- balance amount)) balance)
          "Insufficient funds"))))
//new-withdraw 首先创建一个local 环境，然后返回lambda函数，由于是在local中创建，其closure环境指针指向local,但new-withdraw是在global中创建的         


(define (make-simplified-withdraw balance) (lambda (amount)
(set! balance (- balance amount))
balance))

//(make-simplified-withdraw balance) 等效于new-withdraw 以替代模型理解，不要以传统的call理解，如下python
```

```python
new-withdraw = ((lambda .....) 100)
def make-simplified-withdraw(balance):
	return lambda...
```

<!--more-->

some concepts:

> In contrast to functional programming, programming that makes ex-
> tensive use of assignment is known as imperative programming. 

3.7.scm

```lisp
(define (make-account balance pwd) 
	(define (withdraw amount) 
			(if (>= balance amount)
					(begin (set! balance (- balance amount)) balance)
					"Insufficient funds")) 
	(define (deposit amount)
			(set! balance (+ balance amount))
			balance)

	(define (dispatch m)
		(cond 
			((not (eq? pass password)) (display "wrong password"))
			((eq? m 'withdraw) withdraw)
			((eq? m 'deposit) deposit)
			(else (error "Unknown request: MAKE-ACCOUNT"
			m))))
	(define (pwd-dispatch password)
			(lambda (pass m) 
				(cond 
					((not (eq? pass password)) (display "wrong password") #f)
					((eq? m 'withdraw) withdraw)
					((eq? m 'deposit) deposit)
					((eq? m 'new) (lambda (psw) (pwd-dispatch psw)))
					(else (error "Unknown request: MAKE-ACCOUNT" m))))

			)
	(pwd-dispatch pwd)
	)

(define (make-joint account old new)
	((account old 'new) new)
)
(define a (make-account 100 '222))
(define b (make-joint a '222 '333))
((a '222 'deposit) 40)
((b '333 'withdraw) 30)
```

##### **3.2The Environment Model of Evaluation**

> - A procedure object is applied to a set of arguments by constructing a frame, binding the formal parameters of the procedure to the arguments of the call, and then evaluating the body of the procedure in the context of the new environment constructed. the newframe has as its enclosing environment the environment part of the procedure object being applied.
> - A procedure is created by evaluating a λ-expression relative to a given environment. the resulting procedure object is a pair con-sisting of the text of the λ-expression and a pointer to the envi-ronment in which the procedure was created.

一个python的例子

```python
k=1

def fib():
	return k

def fib2():
	k=11
	def fib3():
		return k
	return fib3(), fib()

print fib2() # (11,1)
```
**3.3	Modeling with Mutable Data**

> (eq? x y) tests whether x and y are the same object (that is, whether x and y are equal as pointers).

​      							**scheme 不区分大小写**	
​		
3.3.4数字模拟电路

功能块为wire添加procedure ，wire触发时procedure的作用是将具体动作加入agenda中，当agenda执行过程中wire值发生变化，wire会将调用procedure列表将具体动作添加到agenda中，agenda会继续执行下去。

**3.4 	Concurrency: Time Is of the Essence**

```lisp
(define (make-account balance) 
  (define (withdraw amount) 
    (if (>= balance amount)
		(begin (set! balance (- balance amount)) balance)
		"Insufficient funds")) 
  (define (deposit amount)
	(set! balance (+ balance amount))
			balance)
  (let ((protected (make-serializer)))
	(define (dispatch m)
		(cond ((eq? m 'withdraw) (protected withdraw))
		      ((eq? m 'deposit) (protected deposit))
          	   ((eq? m 'balance) balance)
			  (else (error "Unknown request: MAKE-ACCOUNT" m))))
dispatch)) //原版


(define (make-account balance)
  (define (withdraw amount)
    (if (>= balance amount)
		(begin (set! balance (- balance amount)) balance)
		"Insufficient funds"))
  (define (deposit amount)
	(set! balance (+ balance amount))
	balance)
	(let ((protected (make-serializer)))
		(let ((protected-withdraw (protected withdraw)) (protected-deposit (protected deposit)))
		(define (dispatch m)
		(cond ((eq? m 'withdraw) protected-withdraw)
		((eq? m 'deposit) protected-deposit)
         ((eq? m 'balance) balance)
		(else (error "Unknown request: MAKE-ACCOUNT" m))))
	dispatch)))
//改进版之后每次调用withdraw protected不会针对每个withdraw和deposit串行化。因为其串化结果已写定,不会分配新的空间，并发的命令会依次进行，会在同一空间调用运行。结果没有变化(可看之后的生成锁理解)
```
```lisp
(define (make-serializer) (let ((mutex (make-mutex)))
(lambda (p)
	(define (serialized-p . args)
		(mutex 'acquire)  //如果mutex已经为#t则会一直执行(mutex 'acquire)
		(let ((val (apply p args)))
          	(mutex 'release)
          	 val))
      serialized-p)))
//创建serializer
```

```lisp
(define (make-mutex)
(let ((cell (list false)))
(define (the-mutex m) 
  (cond ((eq? m 'acquire)
	    	(if (test-and-set! cell) (the-mutex 'acquire))) 
        ((eq? m 'release)
         	(clear! cell))))
    the-mutex))
//生成锁
(define (clear! cell)
	(set-car! cell false))
(define (test-and-set! cell)
	(if (car cell) true (begin (set-car! cell true) false)))

//test-and-set! 必须为原子操作否则可能在set true之前被再次访问
```

代替串行的方式屏障同步？

**3.5 stream**

```lisp
(define (memo-proc proc)
  (let ((already-run? #f)) 
       ((result #f))
       (lambda () 
               (if (not already-run?)
                   (begin (set! already-run #t)
                          (set! result (proc)))
                   result))))
(define (deny func) (memo-proc (lambda() func))) //会生成memo环境保存，memo只会在生成stream的时候调用
(define (force func) (func))
```
素数流

```
(define (divisible? x y) (= (remainder x y) 0))
(define (sieve stream) 
	(cons-stream
         (stream-car stream)
         (sieve 
         	 (stream-filter
			(lambda (x) (not (divisible? x (stream-car stream))))
			(stream-cdr stream)))))
(define primes (sieve (integers-starting-from 2))) //发现一个int ，之后就过滤int的倍数
```
新版本的guess

```lisp
(define (sqrt-improve guess x) 
  (average guess (/ x guess)))

(define (sqrt-stream x) (define guesses
    (cons-stream
     1.0
	(stream-map (lambda (guess) (sqrt-improve guess x)) guesses)))
  guesses)
```

欧拉加速

```
(define (make-tableau transform s)
	(cons-stream s (make-tableau transform (transform s)))
)
//生成多个stream的stream
(define (accelerated-sequence transform s)
	(stream-map stream-car (make-tableau transform s))
)
//取每个stream的第一项组成新的stream
```
```lisp
// stream求导数
(define (integral delayed-integrand initial-value dt) (define int
	(cons-stream
		initial-value
		(let ((integrand (force delayed-integrand)))
         (add-streams (scale-stream integrand dt) int))))  //y(t) = y(0)+f(y)*dt
    int)
//f为f(y)=dy/dt    ,(delay dy) 为了确保define 不出错，比一定适用于所有解释器
(define (solve f y0 dt)
  (define y (integral (delay dy) y0 dt))
  (define dy (stream-map f y))
  y)
  
//ex

(stream-ref (solve (lambda (y) y) 1
  0.001) 1000)
  2.716924
```

3.5.5

```lisp
(define (make-simplified-withdraw balance)
	(lambda (amount)
	(set! balance (- balance amount)) balance))
	
(define (stream-withdraw balance amount-stream) (cons-stream
   balance
   (stream-withdraw (- balance (stream-car amount-stream))
                    (stream-cdr amount-stream))))
// amount-stream 是取钱的金额流
```

> We began this chapter with the goal of building computational mod-
> els whose structure matches our perception of the real world we are
> trying to model. We can model the world as a collection of separate,
> time-bound, interacting objects with state, or we can model the world
> as a single, timeless, stateless unity. Each view has powerful advantages,
> but neither view alone is completely satisfactory. A grand unification
> has yet to emerge.


​			
​		
​	
