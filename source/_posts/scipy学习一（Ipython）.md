---

title: scipy学习一(IPython)
date: 2016-09-18 10:21:05
categories: programming
tags: course
mathjax: true
---

#### 1.Ipython命令行

1.创建一个文件my_file.py

```python
s = 'Hello world'
print(s)
```

2.在ipython可执行此文件，并访问相应的变量（%将脚本变成了函数）

```shell
In [1]: %run my_file.py
Hello world

In [2]: s
Out[2]: 'Hello world'

In [3]: %whos
Variable   Type    Data/Info
----------------------------
s          str     Hello world
```

3.ipython 4 个常用的特性：

<!--more-->

1️历史记录（同UNIX shell）

2️magic function（内置函数即上列%run，ipython默认设置automagic，可以省略%s）

```
In [2]: cd /tmp
/tmp
```

%timeit 测试代码效率

```
In [3]: timeit x = 10 
10000000 loops, best of 3: 39 ns per loop
```

%cpaste 用来复制代码并运行（可保留如命令行的前缀（>>））

```
In [5]: cpaste
Pasting code; enter '--' alone on the line to stop or use Ctrl-D.
:In [3]: timeit x = 10
:--
10000000 loops, best of 3: 85.9 ns per loop
In [6]: cpaste
Pasting code; enter '--' alone on the line to stop or use Ctrl-D.
:>>> timeit x = 10
:--
10000000 loops, best of 3: 86 ns per loop
```

%debug不用多说

```
In [7]: x === 10
  File "<ipython-input-6-12fd421b5f28>", line 1
    x === 10
        ^
SyntaxError: invalid syntax


In [8]: debug
> /.../IPython/core/compilerop.py (87)ast_parse()
     86         and are passed to the built-in compile function."""
---> 87         return compile(source, filename, symbol, self.flags | PyCF_ONLY_AST, 1)
     88

ipdb>locals()
{'source': u'x === 10\n', 'symbol': 'exec', 'self':
<IPython.core.compilerop.CachingCompiler instance at 0x2ad8ef0>,
'filename': '<ipython-input-6-12fd421b5f28>'}
```

（IPython help：

- The built-in IPython cheat-sheet is accessible via the %quickref magic function.
- A list of all available magic functions is shown when typing %magic.）

3️alias

```
In [1]: alias
Total number of aliases: 16
Out[1]:
[('cat', 'cat'),
('clear', 'clear'),
('cp', 'cp -i'),
('ldir', 'ls -F -o --color %l | grep /$'),
('less', 'less'),
('lf', 'ls -F -o --color %l | grep ^-'),
('lk', 'ls -F -o --color %l | grep ^l'),
('ll', 'ls -F -o --color'),
('ls', 'ls -F --color'),
('lx', 'ls -F -o --color %l | grep ^-..x'),
('man', 'man'),
('mkdir', 'mkdir'),
('more', 'more'),
('mv', 'mv -i'),
('rm', 'rm -i'),
('rmdir', 'rmdir')]
```

4️tab补全

```
In [1]: x = 10

In [2]: x.<TAB>
x.bit_length   x.conjugate    x.denominator  x.imag         x.numerator
x.real

In [3]: x.real.
x.real.bit_length   x.real.denominator  x.real.numerator
x.real.conjugate    x.real.imag         x.real.real

In [4]: x.real.
```

