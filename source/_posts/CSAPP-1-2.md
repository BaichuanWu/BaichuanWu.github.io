---
title: CSAPP 1~2
date: 2016-10-24 14:21:59
categories: programming
tags: course
mathjax: true
---

### 系统漫游

#### 主要内容
CSAPP1-2 和 standford 的 编程范式2，3笔记
#### 小计
1.硬件系统组成:总线，I/O设备，主存，CPU
2.操作系统：应用程序和硬件之间的软件；作用：防止硬件被滥用，向应用程序提供简单一致的接口；抽象概念：虚拟机-进程-虚拟存储器-文件（IO)主存)（处理器)操作系统）。
3.并发(concurrency)，杂技员玩球;并行(parallelism),多核。



<!-- more -->


### 信息的表示和处理
#### 主要内容
略
#### 小计
1.字长 整数和指针数据的标称大小

```
//假设short 2字节，int 4 字节
short k=-1;
int v=k;
// v=-1 会将符号位
```

2.大端法(先存高有效位，及高位在低地址先存)和小端法：

<!--more-->

``` bash
#include <stdio.h>
//查看为大端还是小端思路
typedef unsigned char *byte_pointer;
void show_bytes(byte_pointer start, int len){
	int i;
	for(i=0;i<len;i++){
	printf("%.2x", start[i]);
	}
	printf("\n");
}

void show_int(int x){
	show_bytes((byte_pointer) &x, sizeof(int));
}
```
3.无符号数逻辑右移，有符号数算术右移
4.整数四则运算推倒略
5.IEEE浮点标准 $V=(-1)^s\times M\times2^E$ (示例，8位浮点数，1位符号，k=4阶码位，n=3小数位)

|                    类型                    | s(首位，浮点数正负) |        M(尾码，)         |           E(指数)           |
| :--------------------------------------: | :---------: | :-------------------: | :-----------------------: |
|  非规格化 (0~1 0000 000~111 ex:0 0000 011)   |      /      |     M=小数位 ex:3/8      |  $E=1-(2^{k-1}-1)$ ex:-6  |
| 规格化（0~1 0000~1110 000~111 ex:0 0110 110） |      /      | M=小数位+1 ex:6/8+1=14/8 | E=整数位-($2^{k-1}$-1) ex:-1 |
|             无穷(0~1 1111 000)             |      /      |           /           |             /             |
|          NaN(0~1 1111 001~111)           |      /      |           /           |             /             |

6.float浮点数表示

`假设short 2字节 int 4字节 float 4字节`

​	以4字节浮点数为例子。首位(31位)为符号位，30~23位为exp位，剩余为小数位d，从$2^{-1}$开始

​	$f=s\times M\times2^E$		

​	E :-127~128 由8位exp位决定 值为(exp-127)，当exp为0时，为非规格化E=1-127

​	M:M=(1+d)，exp为0时，M=d，

```
int i=5;
float f = i;
printf('%d',f) // 5

float f = * (float *) &i
printf('%d', f) // very small
```
> C语言中的struct的内存满足一下两个原则
>
> 1.结构体变量中成员的偏移量必须是成员大小的整数倍（0被认为是任何数的整数倍）
>
> 2.结构体大小必须是所有成员大小的整数倍。结构体中包含数组数组的大小按其成员大小。

```c
void swap (void *vp1, void * vp2, int size){
  char buffer[size];
  memcpy (buffer, vp1, size);
  memcpy(vp1, vp2, size);
  memcpy(vp2, buffer, size);
}
```
```c
void *lsearch(void *key, void *base, int n, int size){
  for(int i=0;i<n;i++){
    void *elemAddr = (char *) base + i * size
    if (memcmp(elemAddr, key, size)==0) return elemAddr;
  }
  return NULL;
}
```

注意函数指针

```c
void * example(int (*func) (void * base)){};
//传入函数指针
```



```c
typedef struct{
  int logicallen;
  int allocation;
  void * elems；
  int elemSize;
} stack;

void StackNew(stack *s, int elemSize){
  assert(elemSize>0);
  s->logicallen=0;
  s->allocation=4;
  s->elemSize=elemSize;
  s->elems=malloc(4*elemSize);
  assert (s->elems!=NULL);
}

void StackDispose(stack *s){
  free(s->elems);
}
//不能free s 因为不是动态申请的

void StackPush(stack*s, void elemAddr){
  if (s->logicallen==s->allocation){
    StactGrow(s)
    
  }
  void * target=(char *) s->elems + s->logicallen * s->elemSize;
  memcpy(target, elemAddr, s->elemSize);
  s->logicallen++;
}//可以先将s->elems设置为NULL，之后可以用realloc代替malloc，因为不能在原有基础上括展则会重新malloc c++没有直接的realloc
static void StactGrow(stack *s){
  	s->allocation *=2;
    s->elems = realloc(s->elems, s->allocation * s->elemSize);
    assert(s->elems!=NULL);
}//static 函数只能在文件内访问

void stackPop(stack *s, void * elemAddr){
  s->logicallen--;
  void * source=(char *) s->elems + s->logicallen * s->elemSize;
  memcpy(elemAddr, source, s->elemSize);
}
```
> 静态区：保存自动全局变量和static 变量（包括static 全局和局部变量）。静态区的内容在总个程序的生命周期内都存在，由编译器在编译的时候分配。字符串？
> 栈：保存局部变量。栈上的内容只在函数的范围内存在，当函数运行结束，这些内容也会自动被销毁。其特点是效率高，但空间大小有限。
> 堆：由malloc 系列函数或new 操作符分配的内存。其生命周期由free 或delete 决定。

```c
//上段代码升级版


typedef struct{
  int logicallen;
  int allocation;
  void * elems；
  int elemSize;
  void (*func)(void *);
} stack;

void StackNew(stack *s, int elemSize, void (*func)(void * elem)){
  assert(elemSize>0);
  s->logicallen=0;
  s->allocation=4;
  s->elemSize=elemSize;
  s->func = func;
  s->elems=malloc(4*elemSize);
  assert (s->elems!=NULL);
}

void StackDispose(stack *s){
  if (func != NULL){
    (for int i=0;i<s->logicalen;i++){
      s->func((char *) s->elems + i*s->elemSize);  //不在POP中加入free因为返回的东西会立刻清除
    }
  }
  free(s->elems);
}
//不能free s 因为不是动态申请的

void StackPush(stack*s, void elemAddr){
  if (s->logicallen==s->allocation){
    StactGrow(s)
    
  }
  void * target=(char *) s->elems + s->logicallen * s->elemSize;
  memcpy(target, elemAddr, s->elemSize);
  s->logicallen++;
}//可以先将s->elems设置为NULL，之后可以用realloc代替malloc，因为不能在原有基础上括展则会重新malloc c++没有直接的realloc
static void StactGrow(stack *s){
  	s->allocation *=2;
    s->elems = realloc(s->elems, s->allocation * s->elemSize);
    assert(s->elems!=NULL);
}//static 函数只能在文件内访问

void stackPop(stack *s, void * elemAddr){
  s->logicallen--;
  void * source=(char *) s->elems + s->logicallen * s->elemSize;
  memcpy(elemAddr, source, s->elemSize);
}

void stringFree(void * elem){
  free(* (char ** elem)); //注意
}

//memmove(效率低) 和memcpy memmove可以处理有覆盖
```

```c
//指针相减返两个地址之间指针所指数据类型的数量
```

**编程范式8**

内存地址从低到高 汇编代码—>堆区—>栈区

![无奈啊](https://encrypted-tbn3.gstatic.com/images?q=tbn:ANd9GcSd_4yxkCMxQwSQt2S7tBdFw9eTqPS9I_Sl8GD3NarJwmz-bOUkIw)

**堆的实现**

**栈**

SP:STACK POINTER

传入的参数第一个在最下面。（因为在不知道传入参数数量时，能确保访问）

**编译**

1.预处理器（只涉及文本的替换）

```c
#define A 10       
#define B 2*A
//会删除#开头的行，将A替换为10，B替换为2*10
#define MAX(a,b) (((a)>(b))?(a):(b))
//宏，减少函数调用的花费
#ifdef NDEBUG
	#define assert(cond) (void) 0
#else
	#define assert(cond) (cond)?((void) 0):fprintf(stderr, "---"),exit(0)
#endif


#include <stdio.h>
#include "root.h"

//尖括号表明是系统头文件文件 从类似/usr/bin/include /usr/include寻找，""则当前目录由make文件设置
//删除行替换为文件内容
//.h头文件大多只定义原型，不会如.c文件产生机器码，
```
**调试**

```
//c call <malloc>; c++ call <malloc-int-p> c++的函数的重构

//seg fault : 一般对错误指针解引用
//ex:
*(NULL)
//bus error : 解引用的地址不是应当的地址 
//ex:
void *p= xxxxx;
*(short *) p = 7;  //short地址必须为偶地址
*(int *) p=55;	//int 地址必须为4的倍数，除了short 和char 其他数据类型地址都为4的倍数
```
5个科学家吃饭问题，可以设置一个只允许4个人先吃的锁，保证有一个科学家可以先吃，避免死锁