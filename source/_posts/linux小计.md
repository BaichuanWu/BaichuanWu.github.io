---
title: unix | linux小计
date: 2016-09-07 18:06:23
tags: reprinted
---

### 参考UNIX编程环境及相关技术博客知识点小计

##### **1.sh . / . source的区别**

转自[http://www.zengdongwu.com/article3.html]

假如有一个文件test.sh，脚本内容如下

```
#!/bin/bash
echo "step 1 sleeping"
sleep 200
echo "step 2 sleeping"
sleep 200
```

那么，现在按以下4种方式执行：

<!-- more -->

1）./test.sh

2）sh test.sh

3）. test.sh

4）source test.sh

他们有何区别？

1）第一种方式，是在当前的shell执行脚本本身，也就是说把test.sh当成一个文件执行，这时候我们需要拥有test.sh的运行权限（x权限），而且当我们在执行此命令时，有2个新进程在运行，一个是test.sh，一个是sleep，如果我们在执行第一个sleep时按ctrl+c终止脚本，test.sh和sleep一起终止，并且第二个sleep不会执行，因为整个test.sh运行已经终止。

2）第二种方式，是新建一个shell执行test.sh脚本里面的命令，不需要执行权限，有读取权限（r权限）即可，在执行此命令时，有2个新进程在运行，一个是bash，一个是sleep，如果执行第一个sleep时按ctrl+c，bash被终止，结果和第一种方式一样，第二个sleep不会执行。

3）第三种方式，是在当前shell执行test.sh里面的命令，不需要执行权限，有读取权限（r权限）即可，在执行此命令时，只有一个新进程在运行，就是sleep，如果在执行第一个sleep时按ctrl+c终止，那么第二个sleep接着运行，直到脚本所有命令执行完。

4）第四种方式和第三种方式一致。

