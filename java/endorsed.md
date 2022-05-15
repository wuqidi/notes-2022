```-Djava.endrosed.dirs```

指定的目录里面的jar文件，将**覆盖系统API功能**。但出于安全考虑，**不包括java.lang.*包下的类**。

使用场景：jvm的双亲委派机制，jdk提供的类只能由BootClassLoader进行加载，如果需要替换此时可使用endrosed。

除了-D启动命令方式，我们也可以将修改的jar文件放置到```$JAVA_HOME/jre/lib/endorsed```，这样就都生效了。