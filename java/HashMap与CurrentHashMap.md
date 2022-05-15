### HashMap

 * 数据结构：1.7数组+链表 的复合结构 ；1.8数组+链表/红黑树 的复合结构

   hash的值决定了在数组的下标

   ​	hash冲突->hash相同的键值，则以[链表](https://github.com/wuqidi/notes-2022/blob/main/%E7%AE%97%E6%B3%95%E4%B8%8E%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/%E9%93%BE%E8%A1%A8.md)形式存储：
   ​		如果链表size>8，则将链表结构转为[红黑树](https://github.com/wuqidi/notes-2022/blob/main/%E7%AE%97%E6%B3%95%E4%B8%8E%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/%E7%BA%A2%E9%BB%91%E6%A0%91.md)结构；
   ​		如果链表size<6，则红黑树结构转为链表结构；

   ​	hash值的计算：

   ​		如果key不是自定义，是HashMap内部的hash算法
   ​		如果key自定义，需复写hashcode与equals方法

* 扩容：

  * 默认容量16 ```java.util.HashMap#DEFAULT_INITIAL_CAPACITY```

  * 扩容判断条件：负载因子*容量>元素数量

    ​	负载因子=0.75  ，即3/4。代码：```java.util.HashMap#DEFAULT_LOAD_FACTOR```

  
  ​		**为何3/4?**时间和空间上的一种折中方案，从泊松分布上看，0.75，链表大小8，概率会趋近于0.00000006
  
  * 扩容大小：2倍
  
    **为何2倍?**查询下标（n-1)&key.hashCode，n-1的二进制位都是1，可以充分散列，避免不必要的hash冲突。数列均匀，O(1)退化到O(n)。
  
  * 扩容过程：
  
    1.7：数组.foreach->链表.foreach
  
    1.8：只判断原来hash值按位与新数组长度最高位，等于0则原位置；否则原索引+扩容前数组长度
  
* put：1.7头插、1.8尾插

  内部hash算法：

  ```java
      /*
      计算key的hashcode异或该hashcode的高位：这么做的原因是散列表的大小都是2的阶乘，如果不将高位与低位进行异或将会造持续的hash碰撞（当散列表小的时候，高位其实是没用的，会导致大量的hash在低位碰撞）。这样实现的成本也比较低。
      */
  	static final int hash(Object key) {
          int h;
          return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
      }
  
      final V putVal(int hash,// key.hashCode()
                     K key, 
                     V value, 
                     boolean onlyIfAbsent,
                     boolean evict
                    ) {
          Node<K,V>[] tab; Node<K,V> p; int n, i;
          if ((tab = table) == null || (n = tab.length) == 0)
              n = (tab = resize()).length;//初始化 数组[链表] 默认大小是16，n=16
          
          if ((p = tab[i = (n - 1) & hash]) == null)//不存在则新增
              //计算数组下标：(n - 1) & hash
              //(数组长度-1 )& key.hashCode()
              //判断数组该位置是否存在值了
              tab[i] = newNode(hash, key, value, null);
          else {//该数组位置已存在元素
              Node<K,V> e; K k;
              if (p.hash == hash &&
                  ((k = p.key) == key || (key != null && key.equals(k))))
                  //如果key相同
                  e = p;
              else if (p instanceof TreeNode)//是否是红黑树节点
                  e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
              else {//处理碰撞
                  for (int binCount = 0; ; ++binCount) {
                      if ((e = p.next) == null) {
                          p.next = newNode(hash, key, value, null);
                          //新增节点并指向该节点
                          
  /*使用树而不是链表的计数阈值。当向至少有这么多节点的容器中添加元素时，容器将转换为树。该值必须大于2，并且至少应为8，以符合树木移除中关于收缩后转换回普通垃圾箱的假设
      static final int TREEIFY_THRESHOLD = 8; */
                          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                              treeifyBin(tab, hash);//转换红黑树
                          break;
                      }
                      if (e.hash == hash &&
                          ((k = e.key) == key || (key != null && key.equals(k))))
                          //key相同
                          break;
                      p = e;
                  }
              }
              if (e != null) { // key相同 则更新
                  V oldValue = e.value;
                  if (!onlyIfAbsent || oldValue == null)
                      e.value = value;
                  afterNodeAccess(e);//空的 啥也没有，如果有需求可以进行复写
                  return oldValue;
              }
          }
          ++modCount;
          if (++size > threshold)
              resize();
          afterNodeInsertion(evict);//空的 啥也没有
          return null;
      }
  ```

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        /*
        MUST be a power of two <= 1<<30.//已31位
        最大容量，如果构造函数传入的值大于该值，则替换成该值。
        static final int MAXIMUM_CAPACITY = 1 << 30;
        */
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;//32位
            return oldTab;
        }
        /*
        The default initial capacity - MUST be a power of two.
        默认初始容量，必须2的幂
		DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
        */
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // 每次扩容大小是原来2倍
    }
    else if (oldThr > 0) // 初始化容量 threshold
        newCap = oldThr;
    else {               // 初始化 - 无参构造
        newCap = DEFAULT_INITIAL_CAPACITY;//
        /*
        构造函数中未指定时使用的负载系数
         DEFAULT_LOAD_FACTOR = 0.75f;
        */
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;//loadFactor默认是加载因子
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {//遍历数组
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;//数组j下标对象赋给e，并将该节点置为空
                if (e.next == null)//如果e是最后一个元素，则重新计算新数组的下标，将e放置新数组
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)//红黑树
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {//如果元素的hash按位与老数组大小是0
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);//遍历链表
                    /*
                    如果元素的hash按位与老数组大小是0 则原位置
                    否则，新的位置则为原位置+原数组大小
                    */
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

```java
/**
基于指定数组的内容返回哈希代码。如果数组包含其他数组作为元素，则哈希代码基于它们的标识，而不是它们的内容。因此，可以通过一个或多个级别的数组直接或间接地对包含自身作为元素的数组调用此方法。
 */
public static int hashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a)
        result = 31 * result + (element == null ? 0 : element.hashCode());
    return result;
}
//为什么是31？
// 31*(ele1.hash+ele2.hash+...) => (2^5-1)*(ele1.hash+ele2.hash+...)
```

*资料*
[(24条消息) HashMap默认负载因子0.75和泊松分布有关系吗？_掘客DIGGKR的博客-CSDN博客](https://blog.csdn.net/sl1202/article/details/108292693)

### ConcurrentHashMap

* 锁：

  * 早期->分段锁：内部进行分段(**segment**)，数组默认大小还是16个，只锁定当前的元素，避免整体锁定问题。HashEntry内部使用volatile的value来保证可见性，也利用了不可变对象的机制以改进利用unsafe提供的底层能力(例如，volatile access,去直接完成部分操作，以优化性能，毕竟unsafe中的很多操作都是jvm intrinsic优化过的)。
  * 1.8的锁(cas+synchronized)是加在表头(元素)，使用cas操作，cas+volatile=>无锁并发操作，使用unsafe、longadder之类底层优化手段，进行极端情况处理。key是final，value是volatile。依然使用synchronized。
    * longadder：jvm利用空间换时间的方法，利用striped64特性；多数情况下，建议使用atomiclong

* 数据结构：同hashmap1.7/1.8一致

* 扩容：扩容是给段扩容，不是给整体扩容，2倍扩容

  1.7扩容对segment数组中的HashEntry链表进行扩容，然后两个foreach进行迁移

  1.8扩容对数组扩容，把数组划分成多个任务，每个线程执行一个任务扩容

* 比较
```java
//1.7put
/*1.计算segment数组下标位置：(key.hash>>segmentshift)&(数组长度-1)=>数组下标
//segmentshift：检索到段的偏移量
2、判断当前segment是否初始化，如果没有初始化，则通过cas进行初始化
3、计算key的hash，找到在HashEntry位置。
4、由于segment继承了ReentrantLock，所以会尝试获取锁，如果锁成功，将数据插入到HashEntry位置;如果锁冲突则插入到链表末端；如果锁被占用，自旋锁，超过指定次数后挂起，等待唤醒。
*/
//1.7get
/*1、计算segment数组下标位置，找到segment；
2、计算key的hash值，计算HashEntry位置
3、遍历链表，进行匹配
*/
//1.7size
/*1、最多三次计算count，不加锁2次，比较结果，相同则返回；
2、如果2次不加锁无效，则进行加锁，给所有segment加锁，计算count
* 分段锁的副作用，通过两次重试机制，如果没有变化则返回其值；否则加锁进行操作。
*/
```

  ```java
  //1.8put
  /*1、如果没有初始化则进行初始化
  2、如果没有hash冲突，直接cas插入
  3、如果正在进行扩容就先进行扩容，当前线程也会加入进去
  4、如果有hash冲突，则加锁；一种是链表尾部插入；一种插入红黑树
  5、如果链表size大于8则将链表转为红黑树
  6、如果添加成功调用addCount统计size，并检查是否需要扩容；
  */
  //1.8get
  /*1、计算hash，定位到数组下标，如果是首节点则返回
  2、如果正在扩容，会调用正在扩容节点的find方法，查找该节点，并返回；
  3、正常遍历节点，并返回。
  */
  //1.8size
  /*直接返回count*/
  ```

1.7get操作在**大的**HashEntry链表中效率低，1.8get进行了优化采用了红黑树；还有就是优化了size。

1.8put->见下图代码

  ```java
  /**内部hash算法：高位或与运算减少hash冲突，并与全位１，均匀散列。 */
  static final int spread(int h) {
      //static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
      //1111111111111111111111111111111
      return (h ^ (h >>> 16)) & HASH_BITS;
  }
  final V putVal(K key, V value, boolean onlyIfAbsent) {
          if (key == null || value == null) throw new NullPointerException();
      //无法存储 null键和null值
          int hash = spread(key.hashCode());//hash code
          int binCount = 0;
          for (Node<K,V>[] tab = table;;) {//遍历table
              Node<K,V> f; int n, i, fh;
              if (tab == null || (n = tab.length) == 0)
                  tab = initTable();//初始化元素
  /*sun.misc.Unsafe#compareAndSwapInt:cas比较式更新*/
              else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {//如果该下标的元素是空
  /*    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
         return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);}*/
                  if (casTabAt(tab, i, null,
                               new Node<K,V>(hash, key, value, null)))
  /*static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,Node<K,V> c, Node<K,V> v) {
          return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);}*/ 
                      break;                   // no lock when adding to empty bin
              }
              else if ((fh = f.hash) == MOVED)
  /*static final int MOVED     = -1; // 转运节点*/
                  tab = helpTransfer(tab, f);//如果正在调整大小，则进行转运
              else {
                  V oldVal = null;
                  synchronized (f) {//锁
                      if (tabAt(tab, i) == f) {
                          if (fh >= 0) {
                              binCount = 1;
                              for (Node<K,V> e = f;; ++binCount) {
                                  K ek;
                                  if (e.hash == hash &&
                                      ((ek = e.key) == key ||
                                       (ek != null && key.equals(ek)))) {
                                      oldVal = e.val;
                                      if (!onlyIfAbsent)
                                          e.val = value;
                                      break;
                                  }
                                  Node<K,V> pred = e;
                                  if ((e = e.next) == null) {
                                      pred.next = new Node<K,V>(hash, key,
                                                                value, null);
                                      break;
                                  }
                              }
                          }
                          else if (f instanceof TreeBin) {
                              Node<K,V> p;
                              binCount = 2;
                              if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                             value)) != null) {
                                  oldVal = p.val;
                                  if (!onlyIfAbsent)
                                      p.val = value;
                              }
                          }
                      }
                  }
                  if (binCount != 0) {
                      if (binCount >= TREEIFY_THRESHOLD)
                          treeifyBin(tab, i);
                      if (oldVal != null)
                          return oldVal;
                      break;
                  }
              }
          }
          addCount(1L, binCount);
      /*添加到计数，如果表太小且尚未调整大小，则启动传输。如果已经调整大小，则在有工作时帮助执行转移。在换乘后重新检查入住率，看看是否已经需要重新调整入住率，因为调整入住率是滞后的。 */
          return null;
      }
  ```

*资料*

