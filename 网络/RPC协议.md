[OSI模型]: H:\孜孜不倦\学习笔记\github-notes-2022\网络\OSI.md

##### RPC核心组件：

* Client：服务调用方，
* Client Stub：存放服务方的地址，将客户端的请求打包成网络消息，通过网络发送给服务方。
* Server：服务提供方
* Server Stub：存放客户端的消息，解析该消息，并调用本地方法。

##### 流行的RPC框架：

* gRPC：Google开源的，支持跨语言，基于http2.0，底层使用Netty框架。主要是面向移动应用，序列化采用ProtoBuf序列化协议。
* Thrift：Facebook开源的，跨语言，性能比gRPC、dubbo好，比rpcX要逊色。
* Dubbo：阿里开源的，协议、序列化都可以插拔。
* rpcX：GO语言生态的Dubbo。



*资料*

[有了HTTP，为什么还要RPC？ (qq.com)](https://mp.weixin.qq.com/s/AY9Neb6vc9BCKWrRn5UgMw)