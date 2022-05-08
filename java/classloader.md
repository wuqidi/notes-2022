# classloader

jvm启动时，有三个类加载器进行类的加载分别是：

| classloader     | 作用                                       |
| :-------------- | ---------------------------------------- |
| BootClassLoader | JVM的一部分，采用NativeCode实现，没有继承ClassLoader，主要加载位于/jre/lib下面的jar包：rt.jar，并只加载java/javax/sun路径下的类，<br />随着JVM启动后进行核心类加载，并构造ExtensionClassLoader和AppClassLoader两个加载器 |
| ExtClassLoader  | 继承ClassLoader，加载位于/jre/lib/ext目录下的类      |
| AppClassLoader  | 继承ClassLoader，加载位于，负责加载应用程序主函数类          |

这里说的“继承ClassLoader”并不是直接继承而ExtClassLoader、AppClssLoader都是Launcher的内部类，并继承URLClassLoader,URLClassLoader继承SecureClassLoader，SecureClassLoader继承ClassLoader

##### jvm什么时候启动、关闭？

java -jar *.jar 的时候启动，在程序退出后进行关闭

##### classloader加载了啥？

加载jar文件，读取class文件到内存，并将这些静态数据转换为方法区运行时的数据结构，并在堆中生成代表这个类的java.lang.Class实例，这个实例就作为在方法区的访问入口。

```
Class zlass = List.class;
Method[] fields = zlass.getMethods();//没有实例化 但是拿到了所有的 属性或者方法
```

##### 除了加载还做了什么？

加载之后是链接和初始化。
链接：将原始的类定义信息转化到jvm运行，可分为：
​	验证：验证字节码信息是否符合java虚拟机规范，不断的拔出萝卜带出泥的进行class加载
​	准备：创建类或接口中的静态变量，静态变量分配初始值，用于分配所需内存空间
​	解析：常量池的符号引用转为直接引用
初始化：静态变量的赋值动作，静态代码块的执行，父类型的初始化逻辑优先于子类。

##### 常量池的符号引用转为直接引用？

符号引用就是一组符号来描述引用的目标，比如A引用了B类，在class文件中是无法体现B的具体地址的，所以在编译后使用CONSTANT_Class_info的常量表示这个地址，在解析阶段进行替换为真实的地址，即直接引用。符号引用包括：类和接口的全限定名、字段的名称和描述符、方法的名称和描述符。描述符就是public、private...

##### 常量池：

[细说常量池]: https://www.jianshu.com/p/a4e647a25e18	"https://www.zhihu.com/question/55994121"

上面的常量池指的是class常量池，可以理解为一个资源仓库

| 常量池        | 功能                                       |
| ---------- | ---------------------------------------- |
| class常量池   | 每个class文件都有一个class常量池，存放字面量和符号引用         |
| 运行时常量池     | class常量池被加载到堆后的版本，是方法区(8中叫元空间)的一部分，不同的是符号引用转为直接引用；还有区别就是动态性，运行时常量池不只有class常量池内容，运行期间可将新的常量放入其中，例如String.intern()，字面量是字符串的运行时放入字符串常量池中，intern()方法首先判断字符串常量池是否存在equal相等的字符串，有则返回；无则加入字符串常量池。 |
| 基本类型包装类常量池 | 基本类型包装类基本都实现了常量池技术，Byte、Short、Integer、Long默认创建-128~127的缓存数据，Character缓存在 0~127的缓存数据，如果超出范围就新建对象。Float与Double没有实现常量池技术 |
| 字符串常量池     | 字符串常量池没有缓存数据，6之前字符串常量池在方法区，7开始在堆中。<br />方法区or堆影响了intern方法的处理逻辑：6放在方法区和堆中的引用不同；7方法区存储的是堆的引用。6的intern常量池存在则返回常量池引用，否则放入常量池，再返回常量池的引用；7的intern常量池存在则返回常量池引用，否则在常量池放入一个指向原字符串的引用，再返回引用。原来是复制一份副本放入常量池，7之后放入引用。<br />StringTable实现字符串常量池，Hash表结构，默认大小1009,如果放入过大会hash冲突，intern方法挨个遍历。7可指定长度：-XX:StringTableSize=618 |

##### 字面量：

字符串，八种基本类型的值，声明final的常量

##### 啥是双亲委派？

加载时首先判断是否存在父类加载器，如果存在则父类加载，如果父类加载器未能加载该类，则调用findClass进行加载。
保证唯一性、安全性

```
System.out.println(List.class.getClassLoader());//是空 加载器是BootClassLoader
System.out.println(TestLoader.class.getClassLoader());//sun.misc.Launcher$AppClassLoader
System.out.println(TestLoader.class.getClassLoader().getParent());//ExtClassLoader
System.out.println(TestLoader.class.getClassLoader().getParent().getParent());//null
//如果在获取父加载器则空指针，因为已经到Object了，没有了，顶了
```

[线程上下文类加载器：JNDI、JDBC、JCE、JAXB、JBI ***代码热替换与模块热部署（OSGI）](https://github.com/wuqidi/notes-2022/blob/main/java/JNDI%E4%B8%8EOSGI.md)

##### URLClassLoader与SecureClassLoader：

SecureClassLoader：根据系统策略获取到的代码源提供权限支持。一些方法尝试执行特权操作，如果类加载器创建了ProtectionDomain ，则将代码源与当前安全策略匹配，决定是否进行授权。

URLClassLoader：可以加载任意路径下的类，ClassLoader只能加载classpath路径下的类

#### 自定义类加载器

场景：
1、隔离：类似tomcat这类web容器，用于隔离不同应用程序
2、扩展：从网络、数据库等加载类

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();//纳秒级精度的时间
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);//父类加载器
                } else {
                    c = findBootstrapClassOrNull(name);//BootClassLoader
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {//boot和父类都没加载
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);//下一级加载器加载

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}

/**
将class文件加载到内存中，由于该文件是二进制的形式，所以需要defineClass方法将二进制转换为class文件。
*/
protected Class<?> findClass(String name) throws ClassNotFoundException {
    throw new ClassNotFoundException(name);//如果没有重写则抛错
}

protected final Class<?> defineClass(String name, byte[] b, int off, int len,
                                     ProtectionDomain protectionDomain)
    throws ClassFormatError
{
    protectionDomain = preDefineClass(name, protectionDomain);
    String source = defineClassSourceLocation(protectionDomain);
    Class<?> c = defineClass1(name, b, off, len, protectionDomain, source);
    postDefineClass(c, protectionDomain);
    return c;
}
```

继承java.land.ClassLoader，重写loadclass方法(会破坏双亲委派，不推荐)、重写findClass方法

```java
public class QidiClassLoader extends ClassLoader {
    private final String jarPath;

    public QidiClassLoader(ClassLoader parent,String jarPath) {
        super(parent);
        this.jarPath = jarPath;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        BufferedInputStream bis = null;
        ByteArrayOutputStream baos = null;

        String fileName = jarPath + name +".class";
        try {
            bis = new BufferedInputStream(new FileInputStream(fileName));
            baos = new ByteArrayOutputStream();
            int len;
            byte[] data = new byte[1024];
            while ((len=bis.read(data))!=-1){
                baos.write(data,0,len);
            }
            byte[] bytes = baos.toByteArray();
            return defineClass(null,bytes,0,bytes.length);
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                bis.close();
            }catch (Exception e){
                e.printStackTrace();
            }
            try {
                baos.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        //return super.findClass(name);
        return null;
    }
}
```



