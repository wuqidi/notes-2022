# spring-boot

## 起点：JarLauncher

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
      //org.springframework.boot.loader.LaunchedURLClassLoaderj继承URLClassLoader
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

