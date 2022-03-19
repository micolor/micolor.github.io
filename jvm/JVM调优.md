# JVM调优

## top

```shell
top - 01:30:52 up 54 days, 11:11,  1 user,  load average: 0.00, 0.01, 0.05
Tasks:  91 total,   2 running,  89 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.7 sy,  0.0 ni, 98.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1882008 total,    79456 free,  1384372 used,   418180 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   342880 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                 
20670 root      20   0  677196  13616   1664 S  0.7  0.7  21:53.70 barad_agent     
31725 root      20   0  987084  43684   7420 S  0.7  2.3 138:18.49 YDService
```

- 第一行显示的内容：当前时间、系统运行时间以及正在登录用户数
- load average 后的三个数字，依次表示过去 1 分钟、5 分钟、15 分钟的平均负载（Load Average）。 平均负载是指单位时间内，系统处于可运行状态（正在使用 CPU 或者正在等待 CPU 的进程，R 状态）和不可中断状态（D 状态）的平均进程数，也就是平均活跃进程数， CPU 平均负载和 CPU 使用率并没有直接关系。

- 第三行的内容表示 CPU 利用率，每一列的含义可以使用 man 查看。CPU 使用率 体现了单位时间内 CPU 使用情况的统计，以百分比的方式展示。计算方式为：CPU 利 用率 = 1 - （CPU 空闲时间）/ CPU 总的时间。需要注意的是，通过性能分析工具得 到的 CPU 的利用率其实是某个采样时间内的 CPU 平均值。注：top 工具显示的的 CPU 利用率是把所有 CPU 核的数值加起来的，即 8 核 CPU 的利用率最大可以到达 800%（可以用 htop 等更新一些的工具代替 top）。

## vmstat

```
[root]# vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 4  0      0  94620  44620 360036    0    0    39    43    2    4  1  1 98  0  0
 0  0      0  90552  44628 361616    0    0  1572    84  573 1231  3  1 95  1  0
```

上表的 cs（context switch） 就是每秒上下文切换的次数，按照不同场景，CPU 上下文切换还可以分为中断上下文切换、线程上下文切换和进程上下文切换三种，但是无论是哪一种，过多的上下文切换，都会把 CPU 时间消耗在寄存器、内核栈以及虚拟内存等数据的保存和恢复上，从而缩短进程真正运行的时间，导致系统的整体性能大幅 下降。vmstat 的输出中 us、sy 分别用户态和内核态的 CPU 利用率，这两个值也非常 具有参考意义。

## free

```shell
[root ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           1.8G        1.3G         93M        1.0M        392M        334M
Swap:            0B          0B          0B
```
