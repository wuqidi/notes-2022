# Lambda表达式

**格式**：(a,b)->{a.compareTo(b);}
			即：参数列表->lambda主体

**特点**：

* 主体内默认是return；
* return是一个控制流语句，如果显示声明则需要加花括号+分号；
* 借助默认return，可以减少代码量，去除花括号等；是否加花括号取决于控制流语句存在否。

##### lambda使用场景：函数式接口的入参，只为行为参数化

一、函数式接口

```java
@FunctionalInterface
/**
函数式接口：无用指数 ★★★☆☆
	特点：	
	1、接口内只有一个抽象方法，并默认继承Object，也就是说equals、defaul方法、static都不算抽象方法。
	2、该注解是用于表示该接口会设计成一个函数式接口，如果声明了但是接口它不是函数式接口，编译的时候就会报错
	该注解不是必须的，如果接口符合定义是否有注解不影响编译器检查
	应用场景：主要用于行为参数传递，lambda、方法引用，常见于集合操作过程
	两星是确实for循环方便了
*/
// 正确的函数式接口
@FunctionalInterface
public interface TestInterface {
    // 抽象方法
    public void wuhuanzhige();
    // java.lang.Object中的方法不是抽象方法
    public boolean equals(Object var1);
    // default不是抽象方法
    public default void defaultMethod(){
    }
    // static不是抽象方法
    public static void staticMethod(){
    }
}

//函数型接口 Function<T,R> # T:输入类型 R：输出类型
    public static void main(String[] args) {
        Function<String,String> func = s->ss(s);//s->"AI智障请回答："+s
        System.out.println(func.apply("ai wo ca!"));
    }

    public static String ss(String s){//故意给拿出来了，可以简化到上面的注解样子
        return "AI智障请回答："+s;
    }
//结果：实打实的说，应该在stream中有更好的场景，这例子没举好

//判定型接口 Predicate<T>  # T:输入类型 最后输出boolean类型，or、and方法可返回Predicate再下一步处理
    public static void main(String[] args) {
        Predicate<String> predicate = s -> ss(s);
        System.out.println(predicate.test("hahaha"));
    }

    public static boolean ss(String s){
        return StringUtils.isEmpty(s);
    }
//结果：同上

//供给型接口 Supplier<T> #T:输出类型，没入参
 	public static void main(String[] args) {
       ss("wo ca lei!");
    }

    public static void ss(String s){
        Supplier<String> supplier = ()->s;
        System.out.println(supplier.get());
    }
//结果：翻车一晚上了，结论同上

//消费型接口 Consumer<T> #T:输入类型，返回void
//主要用于对类型T元素进行一些操作
    public static void main(String[] args) {
        Consumer<String> consumer = s -> System.out.println(s);
        consumer.accept("这是啥？");
    }
//结果：同上

/**
总结：搞了一晚上发现，搞计算机语言都这么内卷了么？？？！！！语法糖
*/
/*
常见的拆、装箱类型：
IntPredicate/DoublePredicate/IntConsumer/LongBinaryOperator/IntFuntion/ToIntFuntion/IntToDoubleFuntion
*/

```

*<u>**lambda表达式以入参的形式将函数式接口的方法实现**</u>*

![image-20220525231936420](C:\Users\wuqidi\AppData\Roaming\Typora\typora-user-images\image-20220525231936420.png)

![image-20220525232316403](C:\Users\wuqidi\AppData\Roaming\Typora\typora-user-images\image-20220525232316403.png)

***任何函数式接口都不允许抛错***

方法引用：对象::get***();这就是方法引用
构造方法引用：类名::new;这就是构造方法的引用

# 流 Stream

中间操作：返回另一个流，可以让多个操作链接起来形成一个查询；除非触发一个终端操作，否则中间操作不会执行任何一个处理。流的延迟性质。
终端操作：会从中间操作生成结果，其结果不是流。

![image-20220526223020784](C:\Users\wuqidi\AppData\Roaming\Typora\typora-user-images\image-20220526223020784.png)

![image-20220526223039406](C:\Users\wuqidi\AppData\Roaming\Typora\typora-user-images\image-20220526223039406.png)

```java
skip(n)//返回一个扔掉前n个元素的流，不足n个返回空流；与limit互补
distinct()//去重
flatMap()//各个流汇聚成一个流：将流中的每个值都换成一个流，然后把所有的流链接成一个流。
```

##### 5.4 规约



# java8中的安全导航操作符java.util.Optional<T>

```java
public class Man{
    private Girl girlFirendNull;//这里是可能为空的
    private Optional<Girl> girlFriend;
}
Optional<Girl> girlFriend = Optional.empty();//空
Optional<Girl> girlFriend = Optional.of(new Girl());//非空
Optional<Girl> girlFriend = Optional.ofNullable(new Girl()?);//可接受为空
```

如果optional的值是空，由optional.empty返回。

算法、结构、数据

致命问题，Optional无法序列化！！！

![image-20220526233219796](C:\Users\wuqidi\AppData\Roaming\Typora\typora-user-images\image-20220526233219796.png)
