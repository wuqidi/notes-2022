# 看了一眼MANIFEST.MF

事情起因：看了一眼MANIFEST.MF

**字面含义**：“manifest”的意思是“显示”

**为啥要有**？jar命令解析jar包时会查这个文件main-class是谁，本质是定义了jar这种压缩包内部的信息，就像OS启动也需要bootload搭把手，这个手就是启动点。

**功能**：描述jar文件信息

**字段含义说明**：

Manifest-Version: 1.0#manifest的版本

Implementation-Title: DDD-Demo-Server#定义了扩展实现的标题

Implementation-Version: 0.0.1-SNAPSHOT#定义扩展实现的版本

Spring-Boot-Classpath-Index:BOOT-INF/classpath.idx

Spring-Boot-Layers-Index:BOOT-INF/layers.idx

Spring-Boot-Classes: BOOT-INF/classes/

Spring-Boot-Lib: BOOT-INF/lib/

Spring-Boot-Version: 2.6.5
\#以上springboot信息，在JarLauncher启动时使用？解压、启动使用

Start-Class:com.wuqidi.ddddemo.DddDemoApplication#我们的启动类

Build-Jdk-Spec: 1.8#JDK版本号

Created-By: Maven JAR Plugin 3.2.2#该文件的生产者，可以理解为打包工具是哪个

Main-Class: org.springframework.boot.loader.JarLauncher#java-jar 运行的入口

------



## 起点：JarLauncher

Launcher：发射器，引导类

具体用什么launcher取决于构建项目格式：

如果是jar包：org.springframework.boot.loader.JarLauncher

如果是zip包：org.springframework.boot.loader.PropertiesLauncher

如果是war包：org.springframework.boot.loader.WarLauncher

 

JarLauncher不在项目里，怎么看源码？

源码包： 

<!--springboot引导类与项目无关，用于研究源码使用-->

   <dependency>

​       <groupId>org.springframework.boot</groupId>

​       <artifactId>spring-boot-loader</artifactId>

​       <scope>provided</scope>

   </dependency>

 

引导类，本质是类加载器 

Start-Class:com.wuqidi.ddddemo.DddDemoApplication

Main-Class:org.springframework.boot.loader.JarLauncher

```java
//jar调用main方法
    public static void main(String[] args) throws Exception {
       new JarLauncher().launch(args);
    }

	protected void launch(String[] args) throws Exception {
		if (!isExploded()) {//return this.archive.isExploded();//false;JarFileArchive
			JarFile.registerUrlProtocolHandler();
          //注册协议 org.springframework.boot.loader，为后面解析jar使用，可去看java自定义通讯协议
          //org.springframework.boot.loader.jar.Handler
          //org.springframework.boot.loader.jar.JarURLConnection
		  //并且URL.setURLStreamHandlerFactory(null);
		}
		ClassLoader classLoader = createClassLoader(getClassPathArchivesIterator());
      //加载所有的jar包；
      //springboot自己实现ZipFile，JDK默认的AppClassLoader只能从根目录加载文件，并且不支持jar in jar模式 org.springframework.boot.loader.archive.JarFileArchive
      //org.springframework.boot.loader.LaunchedURLClassLoaderj继承java.net.URLClassLoader
		String jarMode = System.getProperty("jarmode");//jarmode
		String launchClass = (jarMode != null && !jarMode.isEmpty()) ? JAR_MODE_LAUNCHER : getMainClass();
      //JAR_MODE_LAUNCHER = "org.springframework.boot.loader.jarmode.JarModeLauncher";
      //getMainClass Start-Class: com.wuqidi.ddddemo.DddDemoApplication
		launch(args, launchClass, classLoader);
      //org.springframework.boot.loader.MainMethodRunner#MainMethodRunner
      //org.springframework.boot.loader.Launcher 作为父类
      /**
      	public void run() throws Exception {
		Class<?> mainClass = Class.forName(this.mainClassName, false,
				Thread.currentThread().getContextClassLoader());
				//Thread.currentThread().setContextClassLoader(classLoader);
				//这里的classloader是LaunchedURLClassLoader
		Method mainMethod = mainClass.getDeclaredMethod("main", String[].class);
		mainMethod.setAccessible(true);
		mainMethod.invoke(null, new Object[] { this.args });
	}
      */
	}
	
//RandomAccessFile https://wenku.baidu.com/view/0dc532f4270c844769eae009581b6bd97f19bc99.html 
//ProtectionDomain https://www.php.cn/manual/view/24766.html

```
整个启动过程从jar到launch，springboot先是注册自己的jar协议，然后解析jar in jar，之后以Launcher作为root进行classloader，最后classloader收尾在LaunchedURLClassLoader，至此加载结束。启动新的线程实例化Start-Class，Class.forName("start-class",false,LaunchedURLClassLoader)，调用main方法。

[java自定义通讯协议](java定义通讯协议.md)

