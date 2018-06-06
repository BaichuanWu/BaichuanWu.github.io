---
title: teocs-4
date: 2016-09-10 20:24:42
categories: programming
tags: course
---

### 从零开始构建现代计算机

#### 四.机器语言

##### tips:

**机器语言：**看作一种约定的形式，利用处理器和寄存器操控内存。

**内存访问：**内存访问命令分为两类：1）算术和逻辑命令。2）load和store命令

**内存寻址方式：**

​	1）直接寻址：直接表示一个指定内存单元的地址或者使用一个符号来表示这个指定的地址；

```
LOAD R1 67 // R1<-Memory[67]
```

​	2）立即寻址：将指令数据域中的内容当做要操作的数据装入寄存器

```
LOADI R1,67 //R1<-67
```

​	3）间接寻址：处理类似指针的语言设施

```
//x=foo[j]
ADD R1, foo,j //R1<-foo+j
LOAD* R2,R1 //R2<-Mem
STR R2,X //X<-R2
```

**HACK计算机：**

<!--more-->

内存地址空间：指令地址空间（只读）；数据地址空间

寄存器：D寄存器（仅用于存储数值）和A寄存器（存储数据和地址）

A指令：二进制表示为(16位)0 v v v  v v v v  v v v v  v v v v 

v-位域表示数字即value值

@value，M为Memory[A]

C指令：二进制表示为(16位)1 1 1 a  c1 c2 c3 c4  c5 c6 d1 d2  d3 j1 j2 j3

a-位域和c-位域表示函数编码，d位域表明结果存储位置(d1表示A寄存器，

d2表示D寄存器，d3表示Memory[A])，j位域表明跳转条件(j1当ALU输出值小于0，跳转，j2为等于0，j3为大于0).

##### 项目

1.Mult.asm

```
// Multiplies R0 and R1 and stores the result in R2.
// (R0, R1, R2 refer to RAM[0], RAM[1], and RAM[2], respectively.)

	@sum
	M=0
	@R0
	D=M
	@count
	M=D
	
	(LOOP)
	@count
	D=M
	@END
	D;JLE
	@R1
	D=M
	@sum
	M=D+M
	@count
	M=M-1
	@LOOP
	0;JMP
	(END)
	@sum
	D=M
	@R2
	M=D
	(DONE)
	@DONE
	0;JMP
	
```

2.Fill.asm

```
// Runs an infinite loop that listens to the keyboard input. 
// When a key is pressed (any key), the program blackens the screen,
// i.e. writes "black" in every pixel. When no key is pressed, the
// program clears the screen, i.e. writes "white" in every pixel.


(RESTART)
@SCREEN
D=A
@0
M=D

(KBDCHECK)
@KBD
D=M
@BLACK
D;JGT
@WHITE
D;JEQ  

@KBDCHECK
0;JMP
(BLACK)
@1
M=-1 
@CHANGE
0;JMP

(WHITE)
@1
M=0 
@CHANGE
0;JMP
(CHANGE)
@1
D=M 

@0
A=M
M=D

@0
D=M+1
M=M+1
@KBD
D=A-D 

@CHANGE
D;JGT 

@RESTART
0;JMP
```