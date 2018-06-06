---
title: linux/unix编程手册-6_10
date: 2018-05-15 11:53:07
categories: programming
tags: tips
---

### linux/unix编程手册-6(进程)

#### 进程和程序

- 程序：

  - 二进制格式标志：最初:a.out,COFF; 现:UNIX实现采用ELF(可执行链接格式)
  - 机器语言指令

  <!-- more -->

  - 程序入口地址
  - 数据
  - 符号表及重定位表
  - 共享库和动态链接信息
  - 其他

- 进程

  - 内核定义的抽象实体，并为该实体分配用以执行程序的各项系统资源
  - 用户内存空间（代码和使用的变量）
  - 一系列内核数据结构构成（维护进程状态的信息）

#### 进程号

```c
#include<unistd.h>

pid_t getpid(void);

pid_t getppid(void);

//pid 的最大值为32767， 一旦分配到32767 计数器会置为300重新分配 
//pid_max:/proc/sys/kernel/pid_max

// /proc/$PID/status

//pstree
```

#### 进程的内存布局

> - 文本段(text)：包含进程运行的程序指令，只读，文本段共享，映射到所有使用进程的虚拟地址空间
> - 初始化数据包段(data)：包含显式初始化的全局变量和静态变量
> - 未初始化数据包段(BSS段)：未初始化的全局变量和静态变量，启动前，本段所有内存初始化0；和前者分开的原因是程序在磁盘存储时，不需要为未初始化的变量分配存储空间
> - 栈
> - 堆

![](https://github.com/BaichuanWu/prictures/raw/master/TLPI/6_1.png)

```c
$ size a.out  
   text    data     bss     dec     hex filename  
   1317     500      24    1841     731 a.out
//       +       +       = 
```

```c
extern char etext, edata, end;
```



#### 虚拟内存

> 程序的局部性
>
> - 空间局部性：程序倾向访问访问过内存地址附近的内存
> - 时间局部性：程序倾向访问已经访问过的内存
>
> 内核为进程分配和释放页表
>
> 进程虚拟地址空间页——页表——RAM页帧
>
> 32位一般虚拟内存每页4K, 64位？

#### 环境变量

> 子进程继承父进程环境变量，并隔离
>
> ```
> /proc/$PID/environ 
> ```
>
> ```c
> #include<stdlib.h>
> 
> extern char **environ;
> 
> char *getenv(const char *name);
>     
> int putenv(char *string);
> // string 指向 'name=value'的字符串  修改string 会影响environ
>     
> int setenv(const char *name, const char *value, int overwrite);
> // 将 name, value 分配至内存缓冲区，之后修改不会影响environ
> 
> int unsetenv(const char *name);
> ```
>
> 环境不仅可以作为进程间通讯；还可以作为程序间通讯的方式

#### 非局部跳转

```c
#include<setjmp.h>

int setjmp(jmp_buf env);

void longjmp(jmp_buf env, int val);

// 调用setjmp 时,把当前进程环境的各种信息，和程序计数器(指向代码)和栈指针寄存器的信息的副本保存到env

register int rvar;
volatile int vvar;
```

### linux/unix编程手册-7(内存分配)

#### 堆内存

> - program break 开始位于 &end

```c
#include <unistd.h>
int brk(void *end_data_segment);
//Returns 0 on success, or –1 on error
//设置 program break 到 end_data_segment 会根据页四舍五入取整

void *sbrk(intptr_t increment);
//Returns previous program break on success, or (void *) –1 on error
//设置 program break 增加 increment
```

```c
#include<stdlib.h>
#include<malloc.h>
void *malloc(size_t size);
//Returns pointer to allocated memory on success, or NULL on error 不初始化内存

void free(void *ptr);
// free一般不会降低program break, 但是当栈顶的空闲内存足够大,free的glibc实现会调用sbrk来降低program break

void *calloc(size_t numitems, size_t size);
//Returns pointer to allocated memory on success, or NULL on error 初始化为0

void *realloc(void *ptr, size_t size);
//Returns pointer to allocated memory on success, or NULL on error 无连续的内存，拷贝至新内存

void *memalign(size_t boundary, size_t size);
//Returns pointer to allocated memory on success, or NULL on error 起始地址参数是boundary的整数倍

```

![](https://github.com/BaichuanWu/prictures/raw/master/TLPI/7_1.png)



![](https://github.com/BaichuanWu/prictures/raw/master/TLPI/7_2.png)

#### 栈内存

```c
#include <alloca.h>
void *alloca(size_t size);
//Returns pointer to allocated block of memory
```



### linux/unix编程手册-8(用户和组)

> Linux 用户ID可以不唯一对应多个用户名

```c
#include <pwd.h>
struct passwd {
        char *pw_name;
        char *pw_passwd;
        uid_t pw_uid;
        gid_t pw_gid;
        char *pw_gecos;
        char *pw_dir;
        char *pw_shell;
};

struct passwd *getpwnam(const char *name);
struct passwd *getpwuid(uid_t uid);
//Both return a pointer on success, or NULL on error; see main text for description of the “not found” case
// 返回的指针指向静态内存，不可重入(not reentrant)


```

```c
#include <grp.h>
struct group {
    char  *gr_name;
    char  *gr_passwd;
	gid_t  gr_gid;
	char **gr_mem;
};
struct group *getgrnam(const char *name);
struct group *getgrgid(gid_t gid);
//Both return a pointer on success, or NULL on error; see main text for description of the “not found” case
// 返回的指针指向静态内存，不可重入(not reentrant)

```

### linux/unix编程手册-9(进程凭证)

> - 实际用户/组ID(real xx id)
>
>   - 进程所属的用户组，子进程继承自父进程
>
> - 有效用户/组ID(effective xx id)
>
>   - 通常有效用户/组ID和实际用户/组ID相同，但是有两种方法导致不同
>
>     - xxx
>
>     - 执行set-user-ID 和 set-group-ID的程序
>
>       - ```shell
>         # ls -l prog
>         -rwxr-xr-x 1 root root 302585 Jun 26 15:05 prog
>         # chmod u+s prog   //Turn on set-user-ID permission bit
>         # chmod g+s prog   //Turn on set-group-ID permission bit
>         # ls -l prog
>         -rwsr-sr-x 1 root root 302585 Jun 26 15:05 prog
>         ```
>
>       - 会将可执行权限x标识改为s
>
>       - 对于shell脚本无效(38.3补充)
>
>   - 有效用户ID=0的进程拥有root权限，称之为特权级进程
>
> - 保存的用户/组ID(saved set-xx-id)
>
>   - 配合set-user-ID 和 set-group-ID使用
>   - 其值为复制的有效用户/组ID
>   - 一些系统调用将标识S的进程的有效用户ID在实际和保存ID间切换
>
> - 文件系统用户/组ID(file-system xx id)
>
>   - linux中对于文件的系统操作的权限是由文件系统ID(结合辅助组ID)决定的
>
> - 辅助组ID

```c
#include<unistd.h>
uid_t getuid(void);
uid_t geteuid(void);
gid_t getgid(void);
gid_t getegid(void);

// 修改有效xxID 调用 ,均返回0:success, -1:fail
int setuid(uid_t uid);
int setgid(gid_t gid);

int seteuid(uid_t uid);
int setegid(gid_t gid);
```

> - 非特权进程调用setuid
>   - 仅能修改有效用户ID，
>   - 且仅能修改为实际的用户ID和保存用户ID
>   - setuid等效于seteuid
> - 特权进程调用以一个非0参数x调用setuid，
>   - 用户的3个id均会变为x，且不再是特权进程，不可逆
>   - 调用seteuid只改变euid，可以以非特权的规则恢复

```c
#include<unistd.h>
// uid, euid为-1是不修改
int setreuid(uid_t ruid, uid_t euid);
int setregid(gid_t rgid, gid_t egid);
```

> - 非特权进程可
>   - 将ruid 设为 当前ruid, euid
>   - 将euid设为 当前ruid, euid, suid
> - 特权进程可
>   - 将ruid设为任意值
>   - 将euid设为任意值
> - 满足以下之一suid会设为euid:
>   - ruid不为-1
>   - euid值不等于原ruid值
> - 注意调用顺序，修改了uid之后gid可能无法修改

类似略

### linux/unix编程手册-10(时间)

> - 真实时间
> - 进程时间(进程创建后使用cpu的时间)
>   - 用户CPU时间
>   - 系统CPU时间

```shell
//系统时区 链接到/usr/share/zoninfo的某文件
$ ll /etc/localtime
lrwxrwxrwx 1 root root 25 5月   2 22:36 /etc/localtime -> ../usr/share/zoneinfo/UTC
```

```shell
$ locale
LANG=en_US.UTF-8
LC_CTYPE=zh_CN.UTF-8
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

```c
//修改环境变量执行
$ LANG=de_DE ./show_time  
```

> 软件时钟(jiffies), linux时间精度取决于jiffies 时钟频率100,250,1000Hz 对应jiffies(10,4,1ms)

