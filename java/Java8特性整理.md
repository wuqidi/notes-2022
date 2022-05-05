Java8特性整理

一、函数式接口

```java
@FunctionalInterface
/**
函数式接口：无用指数 ★★★☆☆
	特点：	
	1、接口内只有一个抽象方法，并默认继承Object，也就是说equals、defaul方法、static都不算抽	象方法。
	2、该注解不是必须的，如果接口符合定义是否有注解不影响编译器检查
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

//供给型接口 Supplier<T> #T:输入类型，返回void
 	public static void main(String[] args) {
       ss("wo ca lei!");
    }

    public static void ss(String s){
        Supplier<String> supplier = ()->s;
        System.out.println(supplier.get());
    }
//结果：翻车一晚上了，结论同上

//消费型接口 Consumer<T> #T:输入类型，返回void
    public static void main(String[] args) {
        Consumer<String> consumer = s -> System.out.println(s);
        consumer.accept("这是啥？");
    }
//结果：我得去学习看书了

/**
总结：搞了一晚上发现，搞计算机语言都这么内卷了么？？？！！！专门给stream配的？？？
*/
//todo 我回头翻翻有案例教材的书看看，补充上去

```