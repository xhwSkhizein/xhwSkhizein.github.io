---
title: Linux Command ulimit 的使用及说明
layout: post
guid: urn:uuid:1bb3128d-938b-460b-8067-98bd25742ff5
tags:
  - 
---

ulimit 

-a：显示目前资源限制的设定；
-c <core文件上限>：设定core文件的最大值，单位为区块；
-d <数据节区大小>：程序数据节区的最大值，单位为KB；
-f <文件大小>：shell所能建立的最大文件，单位为区块；
-H：设定资源的硬性限制，也就是管理员所设下的限制；
-m <内存大小>：指定可使用内存的上限，单位为KB；
-n <文件数目>：指定同一时间最多可开启的文件数；
-p <缓冲区大小>：指定管道缓冲区的大小，单位512字节；
-s <堆叠大小>：指定堆叠的上限，单位为KB；
-S：设定资源的弹性限制；
-t <CPU时间>：指定CPU使用时间的上限，单位为秒；
-u <程序数目>：用户最多可开启的程序数目；
-v <虚拟内存大小>：指定可使用的虚拟内存上限，单位为KB。

```
[root@localhost ~]# ulimit -a
core file size          (blocks, -c) 0           #core文件的最大值为100 blocks。
data seg size           (kbytes, -d) unlimited   #进程的数据段可以任意大。
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited   #文件可以任意大。
pending signals                 (-i) 98304       #最多有98304个待处理的信号。
max locked memory       (kbytes, -l) 32          #一个任务锁住的物理内存的最大值为32KB。
max memory size         (kbytes, -m) unlimited   #一个任务的常驻物理内存的最大值。
open files                      (-n) 1024        #一个任务最多可以同时打开1024的文件。
pipe size            (512 bytes, -p) 8           #管道的最大空间为4096字节。
POSIX message queues     (bytes, -q) 819200      #POSIX的消息队列的最大值为819200字节。
real-time priority              (-r) 0
stack size              (kbytes, -s) 10240       #进程的栈的最大值为10240字节。
cpu time               (seconds, -t) unlimited   #进程使用的CPU时间。
max user processes              (-u) 98304       #当前用户同时打开的进程（包括线程）的最大个数为98304。
virtual memory          (kbytes, -v) unlimited   #没有限制进程的最大地址空间。
file locks                      (-x) unlimited   #所能锁住的文件的最大个数没有限制。
```