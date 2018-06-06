---
title: sicp-4
date: 2016-12-17 14:54:14
categories: programming
tags: course
mathjax: true
---

#### Four.Metalinguistic Abstraction

> - the evaluator, which determines the meaning of epression in a programming language, is just another program.
> - In fact, we can regard almost any program as the evaluator for some language.
> - 求值器（ex:解释器）其实也是另一段程序，所有的程序都可以看作是对某一语言的求值器
> - 一个由自身语言写的求值器被称作自循环直译器（An evaluator that is written in the same language that it evaluates is said to be metacircular）
>
>

**4.1	The Metacircular Evaluator**

<!--more-->

```lisp
//图灵停机问题即是否存在一个FUNC能判断所有的函数P对于输入A是停机还是死循环
//证明假设存在函数(halt? p a)能过判断定义以下函数(能停机则#t)
(define (try p)
	(if (halt? p p)
		(run-foever)
		'halt)
  	)
)
//对于try函数执行(try try)时如果能停机则if 为true 结果函数死循环，反之亦然，所以并不存在halt?函数
```

```lisp
//Y运算符把自己当做自己的参数传进来,真他妈优秀
((lambda (n)
	((lambda (fact) 
             (fact fact n))
	(lambda (ft k) 
            (if (= k 1) 
                1 
                (* k (ft ft (- k 1))))))) 
     10)
```

4.1.7.将求值分为两步，1.生成表达式的的函数。2将函数的参数环境输入求值（ex:即提前确保所有的原eval-if 语句全部转化为原始的if）

##### 4.2 Variations on a Scheme — Lazy Evaluation

> - 正则序（normal order）：normal-order languages delay evaluation of pro-
>   cedure arguments until the actual argument values are needed.
> - 应用序（applictaive order）：the arguments to Scheme procedures are evaluated when the procedure is applied

可以通过修改求值器基本的求值过程使非需求的参数延迟求值

**4.3 	Variations on a Scheme — Nondeterministic Computing**

```lisp
(define (prime-sum-pair list1 list2) (let ((a (an-element-of list1))
        (b (an-element-of list2)))
    (require (prime? (+ a b)))
    (list a b)))
//非确定性求值的基本表示，其不提供具体的解决方法
```
**amb的实现**

```commonlisp

```

**4.4	Logic Programming**


​	
