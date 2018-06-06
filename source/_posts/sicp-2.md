---
title: sicp-2
date: 2016-11-01 15:40:03
categories: programming
tags: course
mathjax: true
---

#### Two.Building Abstractions with Data

2.1.3

```lisp
(define (cons x y) (define (dispatch m)
(cond ((= m 0) x) ((= m 1) y)
(else (error "Argument not 0 or 1: CONS" m)))) dispatch)
(define (car z) (z 0)) (define (cdr z) (z 1))

// exercise 2.4
(define (cons a b)
  (lambda (x) (x a b))
 )
(define (car m)
 	(m (lambda (p q) p) 
)
(define (cdr m)
 	(m (lambda (p q) q)) 
 )
```

map

<!--more-->

```lisp
(define (map proc items)
 	(if (null? items)
        	nil
        (cons (proc (car items))
              (map proc (cdr items)))) 
 )
```

filter

```lisp
(define (filter predicate sequence) (cond ((null? sequence) nil)
        ((predicate (car sequence))
         (cons (car sequence)
(filter predicate (cdr sequence)))) (else (filter predicate (cdr sequence)))))
```

> ​	
> the value of expressing programs as sequence operations is that this
> helps us make program designs that are modular, that is, designs that
> are constructed by combining relatively independent pieces. We can en-
> courage modular design by providing a library of standard components
> together with a conventional interface for connecting the components
> in flexible ways.

**2.2层次性数据和闭包性质**

某种数据对象，通过其组合起的数据对象本身可利用同样的操作在组合。闭包性质使我们建立起层次性的结构。

2.2.3 序列作为一种约定的界面

```lisp
(define (salary-of-highest-paid-programmer records) (accumulate max 0 (map salary (filter programmer? records))))
```

```
(define (accumulate op initial sequence) (if (null? sequence)
      initial
      (op (car sequence)
          (accumulate op initial (cdr sequence)))))
```

用accumulate实现的之前的list方法

```
(define f (lambda (x y . z) ⟨body⟩))
(define (f x y . z) ⟨body⟩)

(define g (lambda w ⟨body⟩))
(define (g . w) ⟨body⟩)
```

**2.3符号数据**

```lisp
(apply func '(a b))

(func 'a 'b) // 与上面表达式相同
```
```lisp
(define (lookup given-key set-of-records) (cond ((null? set-of-records) false)
        ((equal? given-key (key (car set-of-records)))
         (car set-of-records))
(else (lookup given-key (cdr set-of-records)))))   // key map pseudo-code
```

> **Huffman tree**
>
> ​	The algorithm for generating a Huffman tree is very simple. e ideais to arrange the tree so that the symbols with the lowest frequencyappear farthest away from the root. Begin with the set of leaf nodes,containing symbols and their frequencies, as determined by the initialdata from which the code is to be constructed. Now find two leaves withthe lowest weights and merge them to produce a node that has thesetwo nodes as its le and right branches. e weight of the new node isthe sum of the two weights. Remove the two leaves from the originalset and replace them by this new node. Now continue this process. Ateach step, merge two nodes with the smallest weights, removing themfrom the set and replacing them with a node that has these two as itsle and right branches. e process stops when there is only one nodele, which is the root of the entire tree. Here is how the Huffman treeof Figure 2.18 was generated:
>
> Initialleaves {(A8)(B3)(C1)(D1)(E1)(F1)(G1)(H1)}
>
> Merge {(A8)(B3)({CD}2)(E1)(F1)(G1)(H1)}
>
> Merge {(A8)(B3)({CD}2)({EF}2)(G1)(H1)}
>
> Merge {(A8)(B3)({CD}2)({EF}2)({GH}2)}
>
> Merge {(A8)(B3)({CD}2)({EFGH}4)}
>
> Merge {(A8)({BCD}5)({EFGH}4)}
>
> Merge {(A8)({BCDEFGH}9)}
>
> Finalmerge {({ABCDEFGH}17)}

**2.4抽象数据的多重表示**

> The general strategy of checking the type of a datum and calling an
> appropriate procedure is called dispatching on type. is is a powerful
> strategy for obtaining modularity in system design.	

```lisp
(define (tag x) (attach-tag 'rectangular x)) (put 'real-part '(rectangular) real-part) (put 'imag-part '(rectangular) imag-part) (put 'magnitude '(rectangular) magnitude) (put 'angle '(rectangular) angle)
(put 'make-from-real-imag 'rectangular
(lambda (x y) (tag (make-from-real-imag x y))))  // 给生成的对象添加tag
(put 'make-from-mag-ang 'rectangular   
(lambda (r a) (tag (make-from-mag-ang r a))))
'done)  

(apply op (list x y z)) //数组里每一项为参数
```

> 显式分派： 这种策略在增加新操作时需要使用者避免命名冲突，而且每当增加新类型时，所有通用操作都需要做相应的改动，这种策略不具有可加性，因此无论是增加新操作还是增加新类型，这种策略都不适合。
>
> 数据导向：数据导向可以很方便地通过包机制增加新类型和新的通用操作，因此无论是增加新类型还是增加新操作，这种策略都很适合。
>
> 消息传递：消息传递将数据对象和数据对象所需的操作整合在一起，因此它可以很方便地增加新类型，但是这种策略不适合增加新操作，因为每次为某个数据对象增加新操作之后，这个数据对象已有的实例全部都要重新实例化才能使用新操作。



**2.5带有通用型操作的系统**

![2-5MultilayerData](https://github.com/BaichuanWu/prictures/raw/master/sicp/2-5MultilayerData.png)

2.5.2不同类型数据的组合

将其先转换为同种