 JAVA默认提供了对file,ftp,gopher,http,https,jar,mailto,netdoc协议的支持。当我们要利用这些协议来创建应用时，主要会涉及到如下几个类：

​      1.java.net.URL：URL资源

​      2.java.net.URLConnection：各种URL资源连接器

```java
URL url = new URL("http://www.baidu.com");  
URLConnection conneciotn = url.openConnection();
//java.protocol.handler.pkgs https://zhuanlan.zhihu.com/p/84235063
```

新建URL的时候经过getURLStreamHandler获取相关协议，协议存储在Hashtable。
​	如果需要自定义openConnection，就得实现URLStreamHandler子类并加载到URL内部HashTable。

##### 实现方案：

1、实现URLStreamHandlerFactory，通过URL.setURLStreamHandlerFactory设置，但只能被设置一次。
相比系统属性的不同在于，先判断是否实现了该接口，然后才是系统属性，最后在jdk rt.jar中路径查找sun.net.www.protocol.<协议名>.Handler类
注：如果用tomcat、jboss等，这些容器已经注册过了，你实现的方案不会生效。

2、设置java.protocol.handler.pkgs系统属性，要求：
​	实现URLStreamHandler且类名必须Handler，例如 http协议，xx.http.Handler
​	设置启动参数：-Djava.protocol.handler.pkgs=com.wuqidi.协议1|com.wuqidi.协议2

```
   String clsName = packagePrefix + "." + protocol + ".Handler";
   Class<?> cls = Class.forName(clsName);
```

代码参考 https://blog.csdn.net/d_x_program/article/details/75038200