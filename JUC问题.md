#### 1、线程安全单例习题 

单例模式有很多实现方法，饿汉、懒汉、静态内部类、枚举类，试分析每种实现下获取单例对象（即调用getInstance）时的线程安全，并思考注释中的问题 

> 饿汉式：**类加载就会导致该单实例对象被创建** 
>
> 懒汉式：类加载不会导致该单实例对象被创建，而是**首次使用该对象时才会创建** 

实现1：

```java
// 问题1：为什么加 final
// 回答1：防止该类被子类继承，子类覆盖了父类中的一些方法，从而破坏单例

// 问题2：如果实现了序列化接口, 还要做什么来防止反序列化破坏单例
// 回答2：通过readResolve()方法设置反序列化后的返回结果为原来的单例，这样反序列化时就不会把反序列化时字节码生成的对象当成结果返回，而是返回原来的单例
public final class Singleton implements Serializable {
    // 问题3：为什么设置为私有? 是否能防止反射创建新的实例?
    // 回答3：若构造方法Singleton()不设置为私有，就可能被无限的生成对象，就违背了单例模式；不能防止反射创建新的实例，反射的功能强大，它可得到类的构造器对象，更改访问权限，从而创建新的实例
    private Singleton() {}
    // 问题4：这样初始化是否能保证单例对象创建时的线程安全?
    // 回答4：能；静态成员变量的初始化操作在类加载过程中完成，类加载过程由JVM保证代码运行的线程安全，所以类加载阶段对成员变量赋值都是线程安全的
    private static final Singleton INSTANCE = new Singleton();
    // 问题5：为什么提供静态方法而不是直接将 INSTANCE 设置为 public, 说出你知道的理由
    // 回答5：更好的封装性、方便创建对象时做更多的控制操作...
    public static Singleton getInstance() {
        return INSTANCE;
    }
    public Object readResolve() {
        return INSTANCE;
    }
}
```



实现2：

```java
// 问题1：枚举单例是如何限制实例个数的
// 问题2：枚举单例在创建时是否有并发问题
// 问题3：枚举单例能否被反射破坏单例
// 问题4：枚举单例能否被反序列化破坏单例
// 问题5：枚举单例属于懒汉式还是饿汉式
// 问题6：枚举单例如果希望加入一些单例创建时的初始化逻辑该如何做
enum Singleton { 
    INSTANCE; 
}
```



实现3：

```java
public final class Singleton {
    private Singleton() { }
    private static Singleton INSTANCE = null;
    // 分析这里的线程安全, 并说明有什么缺点
    public static synchronized Singleton getInstance() {
        if( INSTANCE != null ){
            return INSTANCE;
        } 
        INSTANCE = new Singleton();
        return INSTANCE;
    }
}
```



实现4：DCL（double-checked-locking）

```java
public final class Singleton {
    private Singleton() { }

    // 问题1：解释为什么要加 volatile ?
    private static volatile Singleton INSTANCE = null;
    
    // 问题2：对比实现3, 说出这样做的意义 
    public static Singleton getInstance() {
        if (INSTANCE != null) { 
            return INSTANCE;
        }
        synchronized (Singleton.class) { 
            // 问题3：为什么还要在这里加为空判断, 之前不是判断过了吗
            if (INSTANCE != null) { // t2 
                return INSTANCE;
            }
            INSTANCE = new Singleton(); 
            return INSTANCE;
        } 
    }
}
```



(推荐的)实现5：

```java
public final class Singleton {
    private Singleton() { }
    // 问题1：属于懒汉式还是饿汉式  懒汉式
    private static class LazyHolder {
        static final Singleton INSTANCE = new Singleton();
    }
    // 问题2：在创建时是否有并发问题   JVM保证其安全性
    public static Singleton getInstance() {
        return LazyHolder.INSTANCE;
    }
}
```

