---
title: linux/unix编程手册-1_5
date: 2018-05-01 11:53:07
categories: programming
tags: tips
---

### linux/unix编程手册-2

#### 基本概念

#### **内核**

##### 内核职责

<!-- more -->

- 进程调度（哪些进程获得CPU的使用，每个进程使用多久）
- 内存管理（虚拟内存管理：进程之间，进程和内核之间相互隔离；只需将进程的一部分内容放在内存中，变相提高CPU使用率）
- 提供文件系统
- 创建终止进程
- 对提供设备访问接口
- 提供系统调用API

> 用户态和核心态
>
> 硬件执行之指令使CPU在两种状态间切换；用户态运行时只能访问用户态内存，操作系统位域内核空间（shell是用户进程）
>
> 文件描述符：实际上是一个索引指向内核为每一个进程所维护的打开的文件记录表

#### 进程

##### 1进程内存布局

- 文本：进程的指令
- 数据：程序静态变量
- 堆：动态分配额外内存
- 栈：（线程仅仅有自己的栈）

##### 2进程的创建

- fork创建，子进程继承父进程数据段，堆段，栈段，在内存中创建副本，共享只读的程序文本
- execve加载新的程序，会销毁现有的文本，数据，堆，栈段

> [特殊的进程](https://blog.csdn.net/gatieme/article/details/51484562)
>
> 0 — idel 进程 (第一个进程，N个处理器N个idle进程，cpu没事干的时候进入)
>
> 1 — init进程
>
> 2 — kthread进程

##### 3内存映射

- 文件映射:将文件部分区域映射到进程的虚拟内存
- 匿名映射:

> 通过传入标志，决定映射的内容其他进程是否可见



### linux/unix编程手册-3

#### 系统编程概念

1. 系统调用流程  (一般:应用程序—>C外壳函数(异常正)—>ststem_call()例程(异常负)—>服务例程(异常正))

> - 通过C函数库的外壳函数发起系统调用
> - 外壳函数将调用参数复制到寄存器，同时将系统调用的编号复制到CPU寄存器(%eax)。
> - 外壳函数执行一条中断指令(int 0x80)，引发处理器从用户态切换到内核态，并执行(0x80)的中断矢量指向的代码。
> - 为响应0x80，内核会调用system_call()列程处理中断
>   - 内核栈保存寄存器值
>   - 审核系统调用编号有效性，参数有效性，执行任务，将结果返回system_call()列程
>   - 从内核栈恢复各寄存器值，将系统的调用返回值置于栈中。
>   - 返回至外壳函数，切换到用户态。
> - 若系统调用返回值表明有误，外壳函数会使用**该值**来设置全局变量errno，然后外壳函数返回调用程序，并同时返回**一个整型值(ex:-1)**，表明是否成功。
>
> tip:
>
> 1. 一些系统调用成功后还是返回-1(ex:getpriority()), 需要在调用前将errno至为0，便于排查(p39)；
>
> 2. ```c
>    void jkstack(void)  __attribute__((noreturn));  
>    // 就是告诉编译器这个函数不会返回给调用者，以便编译器在优化时去掉不必要的函数返回代码。
>    ```
>
>    
>
> ```c
> #define	LINUX_REBOOT_MAGIC1	0xfee1dead 
> #define	LINUX_REBOOT_MAGIC2	672274793
> #define	LINUX_REBOOT_MAGIC2A	85072278
> #define	LINUX_REBOOT_MAGIC2B	369367448
> #define	LINUX_REBOOT_MAGIC2C	537993216
> /*672274793 = 0x28121969
>  85072278 = 0x05121996
> 369367448 = 0x16041998
> 537993216 = 0x20112000*/
> ```
>
> 

### linux/unix编程手册-4 (文件I/O)

#### **I/O相关系统调用**



```c
char buffer[BUFFER_SIZE]
//1
fd = open(pathname,flags,mode)  // flags指定文件打开方式以及是否创建, mode指定创建文件的权限
//2
numread=read(fd, buffer, count) // 从fd中读取至多count字节数据，存储到buffer
//3
numwritten=write(fd,buffer,count) //从buffer读取至多count，写入fd
//4
status=close(fd) //释放fd以及相应内核资源
```

> /proc/$PID/fdinfo 获得系统内任意一进程所打开的fd
>
> ```
>  pos:    0   //当前文件偏移量
>  flags:  0100002
>  mnt_id: 17 // represents mount ID of the file system containing the opened file
> ```
>

```c
// tee.c

#include<unistd.h>
#include<stdio.h>
#include<fcntl.h>
#ifndef BUF_SIZE
#define BUF_SIZE 1024
#endif

int main(int argc, char *argv[]){
	int append = 0;
	int opt;
	while ((opt=getopt(argc, argv, "a"))!=1){
		if ((unsigned char) opt =='a'){ append=1;}
	}
	if (optind>=argc) {
		printf("%s [-a] file\n", argv[0]);
	}
	int fd = open(argv[optind], O_CREAT|O_WRONLY|(append?O_APPEND:O_TRUNC), S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH|S_IWOTH);
	if (fd == -1) {printf("opening file %s", argv[optind]);}
	ssize_t numRead;
	char buf[BUF_SIZE];
	int oFd[2] = {fd, STDOUT_FILENO};
	while ((numRead = read(STDIN_FILENO, buf, BUF_SIZE))>0){
		for (int i=0; i<(int)(sizeof(oFd)/sizeof(oFd[0]));i++){
		if (write(fd, buf, numRead)!=numRead){printf("couldn't wirte whole buff");}
		}
	}
	if (numRead==-1){
		printf("read error");
	}
	if (close(fd)==-1){
		printf("colse error");
	}
	_exit(-1);
}
```



###linux/unix编程手册-5 (深入探究文件I/O) 

#### 文件描述符和打开文件之间的关系

> - 进程级的文件描述符表
>   - 控制文件描述符操作的一组标志(close-on-exec标志)
>   - 打开文件句柄的引用 
> - 系统级的打开文件表
>   - 当前文件偏移量
>   - 打开文件时使用的状态参数flags
>   - 文件访问模式
>   - 信号驱动I/O相关的设置
>   - 对文件i-node的引用
> - 文件系统的i-node表
>   - 文件类型（FIFO ,套接字，常规文件）
>   - 指向文件所持有的锁的列表的指针
>   - 文件的各种属性
>
> **进程A,B的文件描述符指向同一文件句柄：可能是fork()继承或者是UNIX域套接字传递；**
>
> **进程A,B的文件描述符指向不同文件句柄，不同文件句柄指向i-node列表同一条目：对同一文件发起open()调用，（统一进程两次open()同一文件类似）**
>
> ![](https://github.com/BaichuanWu/prictures/raw/master/TLPI/5_2.png)

#### 复制文件描述符

```
./myscripts > t.log 2>&1
```

```c
#include<unistd.h>
#include<fcntl.h>

int newfd;

int dup(int oldfd);

int dup2(int oldfd, int newfd); // 忽略close(newfd)的错误

int fcntl(int fd, int cmd, ...);

//复制时 为 fcntl(oldfd, F_DUPFD, startfd) 大于等于startfd的最小未用值作为描述符编号

//下面三行成功时一致；oldfg不存在时dup2不会close(newfd),new=old时什么都不做
close(2); newfd=dup(1)
newfd=dup2(1,2)
close(2); newfd=fcntl(1, F_DUPFD,2)    
```

```c
#include<unistd.d>
//线程安全read，write
ssize_t pread(int fd, void *buf, size_t count, off_t offset);

ssize_t pwirte(int fd, const void *buf, size_t count, off_t offset);

#include<sys/uio.h>

struct iovec{
    void *iov_base;
    size_t iov_len;
}

//多缓冲区读写

ssize_t readv(int fd, const struct iovec *iov, int iovcnt);

ssize_t writev(int fd, const struct iovec *iov, int iovcnt);
```

#### 非阻塞I/O

> 管道，FIFO，套接字，设备支持非阻塞模式

#### 大文件I/O(LFS)

> 大于2GB (32位)

> 两种方式支持：
>
> - LFS设计的大文件操作API
> - 将宏_FILE_OFFSET_BITS定义为64

#### /dev/fd目录

> 每个进程的/dev/fd 链接到 /proc/$PID/fd
>
> 同时存在/dev/[stdin|stdout|stderr] 分别链接到/dev/fd/[0|1|2]

```c
//对于统一进程以下等价
fd = open("/dev/fd/1", O_WRONLY)
fd = dup(1)
```

#### 临时文件

> 打开文件后立刻unlink