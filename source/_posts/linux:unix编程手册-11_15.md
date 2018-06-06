---
title: linux/unix编程手册-11_15
date: 2018-05-27 11:53:07
categories: programming
tags: tips
---

### linux/unix编程手册-11(系统限制和选项)

> limit.h定义了最小值

```shell
$ getconf OPEN_MAX
7168
$ getconf ARG_MAX
262144
```

略略略

<!-- more -->

### linux/unix编程手册-12(系统和进程信息)

> /proc/$PID

```
/pid/self 访问当前进程 /pid/$PID
/pid/$PID/tasks/$TID 线程
```

![](https://github.com/BaichuanWu/prictures/raw/master/TLPI/12_1.png)

![](https://github.com/BaichuanWu/prictures/raw/master/TLPI/12_2.png)

```c
#include <sys/utsname.h>
int uname(struct utsname *utsbuf)
```

### linux/unix编程手册-13(文件I/O缓冲)

> <u>文件系统的块（补充）</u>
>
> read, write仅在用户缓冲区和内核缓冲区高速缓存(kernel buffer cache)之间复制数据，文件和内核缓冲区的数据复制由内核控制

**用户缓冲区和内核缓冲区高速缓存**

一些控制缓存的函数

```c
#include<stdio.h>

int setvbuf(FILE *stream, char *buf, int mode. size_t size);

void setbuf(FILE *stream, char *buf);

void setbuffer(FILE *stream, char *buf, size_t size);

int fflush(FILE *stream);

//setbuf 忽略return 等价于：
setbuf(fp, buf);
setvbuf(fp, buf, (buf != BULL)? _IOFBF:_IONBF, BUFSIZE);
setbiffer(fp, buf, size);
setvbuffer(fp, buf, (buf != BULL)? _IOFBF:_IONBF, size);


```

setvbuf

- buf不为NULL, buf 和 size决定stream的缓冲区,在堆中分配空间
- buf为NULL, stdio库会为stream自动分配一个缓冲区，不一定使用size值
- mode
  - _IONBF不缓冲，立刻调用write(),open(),stderr默认这种类型
  - _IOLBF行缓冲，对于输出流，换行符和缓冲区满输出，对于输入流，每次读取一行，终端设备默认
  - _IOFBF全缓冲，指代磁盘的流默认此

fflush

- 将输出流的数据刷新到内核缓冲区，无stream会刷新所有stdio缓冲区
- 输入流时，将丢弃缓冲区内数据
- 关闭流时会刷新

> 多数c库实现,stdin和stdout指向同一终端，调用stdin时会隐含一次调用fflush(stdout)

**内核缓冲到磁盘**

> - 同步I/O数据完整性
> - 同步I/O文件完整性（相较于前者，一些不影响文件读取的元数据也需要更新到磁盘上(如文件的时间戳)）

**stdio函数库I/O详图**

![](https://github.com/BaichuanWu/prictures/raw/master/TLPI/13_1.png)

```c
#define _XOPEN_SOURCE 600
#include<fcntl.h>

int posix_fadvise(int fd, off_t offset, off_t len, int advise);
//通知内核优化缓冲区使用的接口，略、、、
```

```c
#include<stdio.h>

int fileno(FILE *stream);

FILE *fdopen(int fd, const char *mode);

//文件流和描述符转换
```

### linux/unix编程手册-14(文件系统)

#### 设备文件

> - 实际设备，虚拟设备
> - 字符型设备，块设备
>   - 字符型设备基于每个字符处理：键盘，终端
>   - 块设备每次处理一块设备，通常为512的倍数：磁盘
> - /dev

**磁盘和分区**

> 磁盘分区可容纳任何类型的信息，但通常只包含之一
>
> - 文件系统
> - 数据区域
> - 交换区域：内核内存管理

#### **文件系统**

![](https://github.com/BaichuanWu/prictures/raw/master/TLPI/14_1.png)

![](https://github.com/BaichuanWu/prictures/raw/master/TLPI/14_2.png)

> 当文件存在大量空白时，文件系统只需将i节点和间接指针块中的相应指针打上0，无需分配额外资源

**VFS**：各种文件系统的抽象层

**日志文件系统**：避免系统崩溃后遍历文件系统的一致性检查(ext3, ext2）

文件的挂载和卸载

```shell
$ mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,size=3995724k,nr_inodes=998931,mode=755)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,mode=755)
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
....
##内容在 /proc/mounts   里面，每个进程都有独立的/proc/$PID/mounts
$ ll /proc/mounts
lrwxrwxrwx 1 root root 11 6月   2 21:25 /proc/mounts -> self/mounts
```

> 绑定挂载和硬链

```shell
$ mkdir d1
$ touch d1/x
$ mkdir d2
$ mount --bind d1 d2
$ ls d2 
x
$ touch d2/y
$ ls d1 
x   y
```

#### **tmpfs**:虚拟内存文件系统



### linux/unix编程手册-15(文件系统)

**获取文件信息**

```c
#include<sys/stat.h>

int stat(const char *pathname, struct stat *statbuf);
int lstat(const char *pathname, struct stat *statbuf);
int fstat(int fd, struct stat *statbuf);

//lstat当访问的是链接时，返回的信息针对符号链接本身


```

