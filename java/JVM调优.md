#### GC类型

1、Minor GC:清理Eden和survivor区，Eden区满了->Minor GC，将对象从Survivor1复制到Survivor2，来回导。

2、Major GC：清理Old区，除了Old区满了，有的时候Minor GC也会触发Major GC。Major GC比Minor GC慢10倍。

3、Full GC:清理整个堆，成本很高。

#### 哪些情况会full GC：

1、老年代无法申请到内存：大对象

避免大对象的产生，尽量让对象在新生代，通过Minor GC进行回收。

2、手动调用System.gc()

System.gc()建议JVM进行Full GC，尽量别用。-XX：+DisableExplicitGC，禁止调用System.gc()。

3、方法区空间不足：需要加载比较多的类

#### 如何排查Full GC

1、jps -ml

2、jstat -gcutil [pid] 2s

3、jmap -histo [pid]  1 more 10

4、jmap -heap [pid] 

5、jmap -F -dump:format=b,file=heapDump [pid] 

 