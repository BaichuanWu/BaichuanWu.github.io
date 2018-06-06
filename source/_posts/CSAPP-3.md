---
title: CSAPP-3
date: 2016-10-24 20:59:48
categories: programming
tags: course
mathjax: true
---

### 程序的机器级表示

#### 主要内容

```
$ gcc -O1 -o p p1.c p2.c
```

-O1 告诉编译器第一级优化，提高优化级别，编译后运行快，编译时间长

```
$ gcc -O1 -S code.c
```

以上会运行编译器，产生一个汇编文件 code.s。

```
$ gcc -O1 -c code.c
```

以上会编译并汇编该代码，生成目标代码文件code.o。

```
$ gobjdump -d code.o
```

以上在mac 下使用gobjdump(具体配置google), linux下为objdump， gobjdump 为反汇编指令。

```
pushl		%ebp
movl		%esp, %ebp

```
**3.4.2 数据传送指令**

<!-- more -->

```
假设%dh=CD ,%eax=98765432  MOV MOVS MOVZ区别 同时在IA32中只有pushl,popl

movb %dh %al         %eax=987654CD
movsbl %dh %al       %eax=FFFFFFCD
movzbl %dh %al       %eax=000000CD
```
> - mov S D 系列操作S,D不能同为内存位置
> - 二元操作如（subq）,D为内存时，S不能为内存
> - 一元操作可以为内存或地址