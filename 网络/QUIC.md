QUIC是一个基于UDP传输协议。QUIC在UDP增加错误处理、可靠性、流控制、内置安全性(TLS1.3)。

实际上QUIC已经将udp实现了大部分tcp的功能，与tcp的区别就在于**quic不是顺序传输数据**。

#### 特点：

* **无队头阻塞**：quic上的多个流之间没有依赖，相互独立。某个流发生丢包，只会影响该流，其他不受影响。
* 灵活性、安全性、减少延迟：

  * quic**切换网络性能提升**：一旦建立quic连接，源和目标IP地址和端口都可以更改，无需重新建立连接。
  * quic**连接是安全的**：quic将tls1.3支持融入到协议中，quic流都经过加密。
  * quic为**udp增加流量控制和错误处理**，并包含重要的安全机制以防止一系列的攻击。
  * quic添加**零往返http请求的支持**：http2需要在浏览器和服务器之间进行多次数据交换才能建立tls会话，然后才能传输数据；quic允许http请求头作为tls握手的一部分。

##### 切换网络怎么做到性能提升的？

TCP的连接标识=源IP+源端口号+目标IP+目标端口号+TCP，其中一个变化则标识变化，需要重新连接。

QUIC的连接标识=服务端产生64位随机数作为ID标识，只要ID不变，连接则不会变化。





*资料*

[(27条消息) QUIC——快速UDP网络连接协议_奇舞周刊的博客-CSDN博客](https://blog.csdn.net/qiwoo_weekly/article/details/123059142)

[为什么需要QUIC | HTTP/3详解 (hungryturbo.com)](https://hungryturbo.com/HTTP3-explained/quic/为什么需要QUIC.html#tcp队头阻塞)









