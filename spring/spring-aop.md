AOP:面向切面编程

方式：通过编译方式的静态代理、运行期间的动态代理

本质：在多个纵向的控制流程中，抽取出相同的子流程形成一个横向的切面。

* 静态代理-AspectJ：编译阶段生成代理类，运行时就是增强后的AOP对象

* 动态代理-SpringAOP：运行时在内存生成AOP对象	
* 静态代理拥有更好的性能优势，但是AspectJ需要特殊的编译处理，而SpringAOP无需特定编译处理

SpringAOP动态代理方式：

* JDK-Proxy：必须有接口，不支持类代理
* CGLib：代理类没有接口，则CGLib代理。可以在运行时动态生成代理的类的子类，是通过继承的方式进行动态代理

Spring使用AspectJ方法：http://c.biancheng.net/spring/annotation-aspectj.html

**JDK-Proxy：**

* 先实现接口InvocationHandler实现代理逻辑

* 用Proxy.newProxyInstance执行代理类

```java
public class MyInvocationHandler implements InvocationHandler {
    //目标对象
    private Object o=null;
	//有参构造
    public MyInvocationHandler(Object o) {
        this.o = o;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    		//此处的o指的是原有功能的实现类的对象（StudentServiceImpl）
         Object  invoke = method.invoke(o,args);//执行原有的功能
        StudentServiceUp.test(); //增加新的功能(若放在method.invoke()前面则在原有功能的前面执行)
        return invoke;
    }
}

public void test(){
        ApplicationContext ac=new ClassPathXmlApplicationContext("spring-config.xml");
        //获取执行原有功能的对象
        StudentService studentServiceImpl = (StudentService) ac.getBean("studentServiceImpl");
        //以上两步是用spring集成mybatis 所实现的对象的创建（可以直接用new的方式去获取）
        //创建InvocationHandler对象
        InvocationHandler invocationHandler=new MyInvocationHandler(studentServiceImpl);
        //使用jdk携带的proxy进行动态代理 (参数1：原有功能对象的类 参数2：原有功能对象类所实现的接口 参数3：代理对象要执行的功能)
       StudentService proxy= (StudentService) Proxy.newProxyInstance(studentServiceImpl.getClass().getClassLoader(),
                studentServiceImpl.getClass().getInterfaces(),invocationHandler);
       //测试 执行升级后的test
       proxy.test();
    }
```
*资料*

[彻底理解Spring AOP_慕城南风的博客-CSDN博客_springaop](https://blog.csdn.net/lovedingd/article/details/108026587)

[Spring常见面试题总结（超详细回答）_张维鹏的博客-CSDN博客_spring面试题](https://blog.csdn.net/a745233700/article/details/80959716?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase)

[CGLIB全网详细教程 - 简书 (jianshu.com)](https://www.jianshu.com/p/cbd4c1ad8a75)