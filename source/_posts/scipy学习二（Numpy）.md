---
title: scipy学习二（Numpy）
date: 2016-09-18 10:57:25
categories: programming
tags: course
mathjax: true
---

### Numpy

#### 1.Numpy array object

**1.numpy比python的array类对象比较**

```
In [1]: L = range(1000)

In [2]: %timeit [i**2 for i in L]
1000 loops, best of 3: 403 us per loop

In [3]: a = np.arange(1000)

In [4]: %timeit a**2
100000 loops, best of 3: 12.7 us per loop
```

```python
>>> b = np.array([[0, 1, 2], [3, 4, 5]])
>>> c=b.tolist()
>>> c
[[0, 1, 2], [3, 4, 5]]
```

<!--more-->

**2.array对象相关方法**

```python
>>> b = np.array([[0, 1, 2], [3, 4, 5]])    # 2 x 3 array
>>> b
array([[0, 1, 2],
       [3, 4, 5]])
>>> b.ndim
2
>>> b.shape
(2, 3)
>>> len(b)     # returns the size of the first dimension
2
>>>b.shape=(3,2) #可以改变维度但总数必须不变
>>>b2=b.reshape(3,2)

>>> c = np.array([[[1], [2]], [[3], [4]]])
>>> c
array([[[1],
        [2]],

       [[3],
        [4]]])
>>> c.shape
(2, 2, 1)

>>>
```

**2.生成array的方法**

```python
>>> a = np.arange(10) # 0 .. n-1  (!)
>>> a
array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> b = np.arange(1, 9, 2) # start, end (exclusive), step
>>> b
array([1, 3, 5, 7])
```

```python
>>> c = np.linspace(0, 1, 6)   # start, end, num-points
>>> c
array([ 0. ,  0.2,  0.4,  0.6,  0.8,  1. ])
>>> d = np.linspace(0, 1, 5, endpoint=False)
>>> d
array([ 0. ,  0.2,  0.4,  0.6,  0.8])
```

```python
>>> a = np.ones((3, 3))  # reminder: (3, 3) is a tuple 默认float
>>> a
array([[ 1.,  1.,  1.],
       [ 1.,  1.,  1.],
       [ 1.,  1.,  1.]])
>>> b = np.zeros((2, 2))
>>> b
array([[ 0.,  0.],
       [ 0.,  0.]])
>>> c = np.eye(3)
>>> c
array([[ 1.,  0.,  0.],
       [ 0.,  1.,  0.],
       [ 0.,  0.,  1.]])
>>> d = np.diag(np.array([1, 2, 3, 4]))
>>> d
array([[1, 0, 0, 0],
       [0, 2, 0, 0],
       [0, 0, 3, 0],
       [0, 0, 0, 4]])
```

*随机生成*

```python
>>> a = np.random.rand(4)       # uniform in [0, 1]
>>> a  
array([ 0.95799151,  0.14222247,  0.08777354,  0.51887998])

>>> b = np.random.randn(4)      # Gaussian 正态分布
>>> b  
array([ 0.37544699, -0.11425369, -0.47616538,  1.79664113])

>>> np.random.seed(1234)        # seed是伪随机数random开始计算的起始值(seed 一样，后面的随机值和顺序都一样，一般seed可取真随机数，如当前时间？)
```
**3.array切片操作**

1️一维数组切片同python list对象

```python
>>> a = np.arange(10)
>>> a[5:] = 10  # 比原生强大
>>> a
array([ 0,  1,  2,  3,  4, 10, 10, 10, 10, 10])
>>> b = np.arange(5)
>>> a[5:] = b[::-1]
>>> a
array([0, 1, 2, 3, 4, 4, 3, 2, 1, 0])
```

2️二维

```python
>>>a=np.array([range(i*10,i*10+6) for i in range(6)])
>>>a[0,3:5]   # 第一行，四五列，二维及用逗号隔开一维的切片操作
array([3,4])
>>>a=np.arange(6) + np.arange(0, 51, 10)[:, np.newaxis] # np生成二维数组的另一种方法
```

**4.array的填充操作**

1️np.tile 对整个数组重复

```
>>> a=np.array([10,20])  
>>> np.tile(a,(3,2)) #构造3X2个copy  
array([[10, 20, 10, 20],  
       [10, 20, 10, 20],  
       [10, 20, 10, 20]])  
>>> np.tile(42.0,(3,2))  
array([[ 42.,  42.],  
       [ 42.,  42.],  
       [ 42.,  42.]])  
```

2️np.repeat 数组内每个对象重复

```python
>>>a=np.arange(10)
>>> a.repeat(5)
array([0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 2, 2, 2, 2, 2, 3, 3, 3, 3, 3, 4, 4, 4,4, 4, 5, 5, 5, 5, 5, 6, 6, 6, 6, 6, 7, 7, 7, 7, 7, 8, 8, 8, 8, 8, 9,9, 9, 9, 9])
>>> a=np.array([10,20])  
>>> a.repeat([3,2])  
array([10, 10, 10, 20, 20]) 
```

**5.array的复制操作**

```python
# 原生的list对象切片操作时会自动复制并赋值 （右值）
>>> a=range(10)
>>> b=a[:3]
>>> b[0]=100
>>> a
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> b
[100, 1, 2]

# numpy的array 则共享内存 (左值)
>>> a=np.arange(10)
>>> b=a[:3]
>>> b[0]=100
>>> a
array([100,   1,   2,   3,   4,   5,   6,   7,   8,   9])
>>> b
array([100,   1,   2])
>>> np.may_share_memory(a,b)
True
```

```python
# nonezero返回为True的坐标(tuple中每个数组表示一个维度)
>>> x = np.eye(3)
>>> x
array([[ 1.,  0.,  0.],
       [ 0.,  1.,  0.],
       [ 0.,  0.,  1.]])
>>> np.nonzero(x)
(array([0, 1, 2]), array([0, 1, 2]))

>>> x[np.nonzero(x)]
array([ 1.,  1.,  1.]) # 见后面index操作

>>> a = np.array([[1,2,3],[4,5,6],[7,8,9]])
>>> a > 3
array([[False, False, False],
       [ True,  True,  True],
       [ True,  True,  True]], dtype=bool)
>>> np.nonzero(a > 3)
(array([1, 1, 1, 2, 2, 2]), array([0, 1, 2, 0, 1, 2]))

>>> (a > 3).nonzero()
(array([1, 1, 1, 2, 2, 2]), array([0, 1, 2, 0, 1, 2]))
```

**6.array的mask和index操作**

```python
>>> np.random.seed(3)
>>> a = np.random.random_integers(0, 20, 15)
>>> a
array([10,  3,  8,  0, 19, 10, 11,  9, 10,  6,  0, 20, 12,  7, 14])
>>> (a % 3 == 0)
array([False,  True, False,  True, False, False, False,  True, False,
        True,  True, False,  True, False, False], dtype=bool)
>>> mask = (a % 3 == 0)
>>> extract_from_a = a[mask] # or,  a[a%3==0]
>>> extract_from_a           # extract a sub-array with the mask
array([ 3,  0,  9,  6,  0, 12])

>>> a[a % 3 == 0] = -1
>>> a
array([10, -1,  8, -1, 19, 10, 11, -1, 10, -1, -1, 20, -1,  7, 14])
```

```python
>>> a = np.arange(0, 100, 10)
>>> a
array([ 0, 10, 20, 30, 40, 50, 60, 70, 80, 90])
>>> a[[2, 3, 2, 4, 2]]  # note: [2, 3, 2, 4, 2] is a Python list
array([20, 30, 20, 40, 20])

>>> a[[9, 7]] = -100
>>> a
array([   0,   10,   20,   30,   40,   50,   60, -100,   80, -100])

# 如果一个array被另一个array对象索引，其自动转换为索引的shape
>>> a = np.arange(10)
>>> idx = np.array([[3, 4], [9, 7]])
>>> idx.shape
(2, 2)
>>> a[idx]
array([[3, 4],
       [9, 7]])
```

#### 2.np.array的运算

```python
>>> a = np.array([1, 2, 3, 4])
>>> a + 1
array([2, 3, 4, 5])
>>> 2**a
array([ 2,  4,  8, 16])

>>> b = np.ones(4) + 1
>>> a - b
array([-1.,  0.,  1.,  2.])
>>> a * b
array([ 2.,  4.,  6.,  8.])

>>> j = np.arange(5)
>>> 2**(j + 1) - j
array([ 2,  3,  6, 13, 28])
```

只是数组中每个元素之间的运算，不是矩阵运算

```python
>>> c = np.ones((3, 3))
>>> c * c                   # NOT matrix multiplication!
array([[ 1.,  1.,  1.],
       [ 1.,  1.,  1.],
       [ 1.,  1.,  1.]])

>>> c.dot(c)
array([[ 3.,  3.,  3.],
       [ 3.,  3.,  3.],
       [ 3.,  3.,  3.]])
```

比较运算

```python
>>> a = np.array([1, 2, 3, 4])
>>> b = np.array([4, 2, 2, 4])
>>> a == b
array([False,  True, False,  True], dtype=bool)
>>> a > b
array([False, False,  True, False], dtype=bool)

>>> a = np.array([1, 2, 3, 4])
>>> b = np.array([4, 2, 2, 4])
>>> c = np.array([1, 2, 3, 4])
>>> np.array_equal(a, b)
False
>>> np.array_equal(a, c)
True
```

布尔运算

```python
>>> a = np.array([1, 1, 0, 0], dtype=bool)
>>> b = np.array([1, 0, 1, 0], dtype=bool)
>>> np.logical_or(a, b)
array([ True,  True,  True, False], dtype=bool)
>>> np.logical_and(a, b)
array([ True, False, False, False], dtype=bool)
```

黑科技

```python
>>> a = np.arange(5)
>>> np.sin(a)
array([ 0.        ,  0.84147098,  0.90929743,  0.14112001, -0.7568025 ])
>>> np.log(a)
array([       -inf,  0.        ,  0.69314718,  1.09861229,  1.38629436])
>>> np.exp(a)
array([  1.        ,   2.71828183,   7.3890561 ,  20.08553692,  54.59815003])
```

转置矩阵

