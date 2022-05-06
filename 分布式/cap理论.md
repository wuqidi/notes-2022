CAP：Consistency 一致性，Availability 可用性 ，Partition tolerance 分区容忍性
一致性：强调数据正确性，每个客户端返回的值**要绝对一致并最新**  要么最新要么失败，不是最新则失败
可用性：强调请求不出错不阻塞，每次请求都会返回数据，无法保证是最新数据
分区容忍性：分区之间数据不同步，不挂掉

[CAP 定理的含义 - 阮一峰的网络日志 (ruanyifeng.com)](http://www.ruanyifeng.com/blog/2018/07/cap.html)