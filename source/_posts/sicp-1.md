---
title: sicp-1
date: 2016-10-20 18:08:34
categories: programming
tags: course
mathjax: true
---

#### One.Building Abstractions with Procedures

1.1.5.应用程序替代模型tip:解释器运行的两种方式(顺序)

- ```lisp
  (define (<name> <paramaters>)
    <body>)
  (define (square x) (* x x))
  ```


- **applicative order** 递归，从最外层开始将所有参数求值 (lisp使用)，相交于后者避免重复运算
- **normal order** 全部展开直到所有操作符为原始，再做运算

1.1.6.条件语句

- ```lisp
  (cond (<p1> <e1>)
    	  (<p2> <e2>)
    	  ...)

  (cond (<p1> <e1>)
    	  (else <e2>))

  (if <oredicate> <consequent> <alternative>)
  ```

1.2 迭代和递归（递归过程：1递归计算过程；2迭代计算过程）

<!--more-->

- this type of process, charaterized by a chain of deferred operations, is called a recursive process（递归计算过程,需要track 直到 开始计算):wq

- iterative process(迭代计算过程， 用**固定数量**变量标识计算过程，并有规则描述状态变化时变量的更新方式)

- 尾递归（如C语言对于迭代过程的递归计算，消耗的内存和调用的数目成正比，而scheme实现则进行了优化，只会在常量空间进行计算，这种实现称为尾递归）

- example (O(logn))step的Fib

  ```lisp

  (define (even? n)
  		(= (remainder n 2) 0)
  )

  (define (fib n)
  	(fib-iter 1 0 0 1 n)
  )
  (define (fib-iter a b p q count)
  	(cond ((= count 0) b)
  		  ((even? count) (fib-iter a
  								   b
  								   (+ (* p p) (* q q)) 
  								   (+ (* 2 p q) (* q q))
  								   (/ count 2)
  						))
  		  (else (fib-iter (+ (* b q) (* a q) (* a p))
  						  (+ (* b p) (* a q))
  						  p
  						  q			
  						  (- count 1)
  				))
  	)
  )
  (fib 10)
  ```

- **lame定理**：欧几里得算法需要用K部计算的一对整数的GCD（最大公约数），那么这对整数的较小值必然大于fib(k)

  简要证明：GCD(a1,b1)—>GCD(a2,b2)—>GCD(a3,b3)必有b1>=b2+b3 以此类推

  lame可得出欧几里得算法下的GCD增长阶为O(log n)


- **费马小定律**：如果p是一个素数，则$a^p$(mod p) == a(mod p) 

- > 运行时间，步数和算法复杂度


- ```lisp
  (define (expmod base exp m) (cond ((= exp 0) 1)
          ((even? exp)
           (remainder (* (expmod base (/ exp 2) m)
                         (expmod base (/ exp 2) m))
                      m))
  (else
  (remainder (* base
                         (expmod base (- exp 1) m))
                      m))))
  //如果用* 而不是sqrt 则每个(expmod base (/ exp 2) m)相同值会计算两次
  ```


- **费马小定律推论**：$a^{p-1}$(mod p) = 1 (p为素数，n>1)

- > 证明：假设有一种由p个珠子组成的项链，项链中珠子的种数为最多为a，那么所有可能的不重复的排列为$a^p​$，其中种数为1的情况有a种，那么去除这些情况后一共就有$a^p​$− a种。现在吧所有情况的项链首尾相接，那么就有可能出现重复的情况了。因为p是素数，所以一个包含着p个珠子的环旋转后会出现p种不同的情况（这里就不证明了，即使不是像1+1=2那么显而易见，但也是很容易证明的），亦即，这$a^p​$− a种情况可以分为一些类，每类有p中情况，亦即，$a^p​$ − a 能被p整除


- **Miller-Rabin算法**

- > 若 p是素数，a 是小于 p的正整数，且 $a^2$ mod p=1，那么要么 a=1，要么 a=p−1。

  可以将p-1 = $2^k$*d，不断$^2{2^k d}$ (mod p) 直到发现 不为 1和-1的 m，取$m^2$ (mod p)

  1.3.2

  ```lisp
  (define plus4 (lambda (x) (+ x 4)))
  (define (plus4 x) (+ x 4))  //相同

  (let ((⟨var1⟩ ⟨exp1⟩) (⟨var2⟩ ⟨exp2⟩)
  ...
  (⟨varn⟩ ⟨expn⟩)) ⟨body⟩) // 仅在body中var有效

  (lambda(⟨var1⟩ ... ⟨varn⟩) ⟨body⟩)
  ⟨exp1⟩ ... ⟨expn⟩)  //和let语句等效
  ```


**1.33 procedure as arguments**

**1.34 procedure as returned values**

