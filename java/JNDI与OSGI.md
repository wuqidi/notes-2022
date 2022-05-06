##### JNDI:java命名和目录接口

两部分：API （应用程序编程接口）、SPI （服务供应商接口）

api提供了java应用程序访问各种命名和目录服务的功能

spi提供任意一种服务的供应商提供的功能

<u>有点像微服务的服务的使用方和提供方。</u>

##### OSGI：开放服务网关协议，面向java的动态模型系统

OSGI框架从概念上可以分为三层：

Module Layer：模块层主要涉及包及共享的代码；

Lifecycle Layer：生命周期层主要涉及组件的运行时生命周期管理；

Service Layer：服务层主要涉及模块之间的交互和通信。

资料[架构设计——OSGI规范-java (uml.org.cn)](http://www.uml.org.cn/j2ee/202104022.asp)

