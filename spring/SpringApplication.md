```java
public ConfigurableApplicationContext run(String... args) {
   long startTime = System.nanoTime();
   DefaultBootstrapContext bootstrapContext = createBootstrapContext();
   ConfigurableApplicationContext context = null;
   configureHeadlessProperty();
   SpringApplicationRunListeners listeners = getRunListeners(args);
    //运行监听器SpringApplicationRunListener.class
   listeners.starting(bootstrapContext, this.mainApplicationClass);
    //监听器开启topic:spring.boot.application.starting
   try {
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
      ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);//配置环境
      configureIgnoreBeanInfo(environment);
      Banner printedBanner = printBanner(environment);//打印logo
      context = createApplicationContext();//创建上下文
      context.setApplicationStartup(this.applicationStartup);
      prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);//准备上下文
      refreshContext(context);//刷新上下文
      afterRefresh(context, applicationArguments);//
      Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), timeTakenToStartup);
      }
      listeners.started(context, timeTakenToStartup);
      callRunners(context, applicationArguments);
   }
   catch (Throwable ex) {
      handleRunFailure(context, ex, listeners);
      throw new IllegalStateException(ex);
   }
   try {
      Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
      listeners.ready(context, timeTakenToReady);
   }
   catch (Throwable ex) {
      handleRunFailure(context, ex, null);
      throw new IllegalStateException(ex);
   }
   return context;
}
```

##### java.awt.headless：true->缺少显示屏、鼠标、键盘等

##### ConfigurableEnvironment 

##### ConfigurableApplicationContext

##### WebApplicationType

资料：

[(24条消息) 从0-1了解SpringBoot如何运行（一）：Environment环境装配_原来是笑傲菌殿下的博客-CSDN博客](https://blog.csdn.net/Laugh_xiaoao/article/details/123982865)
