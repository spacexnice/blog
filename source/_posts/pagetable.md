title: 内核页表的传递
date: 2016-01-27 09:24:44
tags: ["页表"]
categories: ["操作系统","Linux内核"]
---
## 问题一：
每个用户进程都有自己的页全局目录及页表，然后内核代表进程在内核态执行，此时如果内核代码修改了内核页表，那么这些修改是如何传递到其他用户进程的？毕竟所有用户进程维护自己的页表，同时关于内核线性空间的页表还必须相同，这个是如何做到的？

<!-- more -->

答案是:

map\_vm\_area()并不触及当前进程页表，而是直接修改init进程页表也就是主内核页表。一旦内核代表当前进程访问非连续内存区是，缺页发生，缺页处理程序会检查该地址是否是内核地址，并且该线性地址是否在主内核页表中。一旦处理程序发现一个主内核页表的有一个该线性地址的非空项，则将该项拷贝到当前进程的页表中。

## 问题二：
那么问题又来了，第一次内核访问非连续存储区的时候，由于进程页表的相应项为空会发生缺页异常，但是如果此后主内核页表项被修改，然后内核再次代表该进程访问非连续物理内存区的时候，由于页表已经被拷贝过旧的值，因此不会发生缺页异常，因此就会访问都错误的非连续的内存区，是这样么？

答案是：

因为在上一步的页表拷贝中，只拷贝了页全局目录的部分项，因此当前进程与主内核页表共享相同的页表项，并且这些页表项一旦被分配就不会被回收，但可以修改。这样所有对主内核页表的修改都会传递到当前进程。如果进程试图访问一个已经被释放发非连续内存区也会引发缺页异。

参见《深入理解Linux内核》

