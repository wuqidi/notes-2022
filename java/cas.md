CAS 的底层是 lock cmpxchg 指令（X86 架构），在单核 CPU 和多核 CPU 下都能够保证【比较-交换】的原子性

在多核状态下，某个核执行到带 lock 的指令时，CPU 会让总线锁住，当这个核把此指令执行完毕，再开启总线。这个过程中不会被线程的调度机制所打断，保证了多个线程对内存操作的准确性，是原子的。

*资料*

[(24条消息) Java无锁并发详细教程_夏小白.的博客-CSDN博客_java无锁并发](https://blog.csdn.net/xia1140418216/article/details/121007970)

[并发编程之无锁 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1587913)