---
title: '[设计模式]微信公众号获取access_token的单例模式'
date: 2019-05-05 18:10:40
tags:
- 设计模式
categories:
- 设计模式
- 创建型模式
---

在我们的系统中，有一些对象我们只需要一个即可，比如线程池或者唯一设备驱动对象等。换句话说，这一类的对象只能有一个实例，否则会有多实例造成的对象状态不一致现象或者资源使用过量等后果，这时候我们可以使用单例模式。笔者从学校接手过微信公众号后台开发的工作，其中从微信获取access_token并存储的这部分就是只需要一个对象即可，笔者将用此文来解说单例模式的实际应用与多种方式的实现。

<!-- more -->

### 定义

{% note default %} 
确保一个类只有一个实例，并提供一个全局访问点。
{% endnote %}

![单例模式类图.png](https://i.loli.net/2019/05/21/5ce3ec33de13436007.png)

**注意：单例模式的类也可以是一般的类，具有一般的属性和方法。**


### 单例模式的实现

单例模式有多种实现方式，笔者认知范围内，共有七种，它们有各自的特点：延迟/非延迟加载，线程安全/非线程安全等，下面我们一个一个介绍。

#### 饿汉式

饿汉式就是指全局的单例实例在类装载时构建，即非延迟加载，或者称为急切实例化。

饿汉式在类创建的同时就已经创建好一个静态的对象供系统使用,以后不再改变，则无须关注多线程问题，并且写法简单明了。

##### 普通饿汉式(线程安全)

```java
public class Singleton {  
    private static final Singleton INSTANCE = new Singleton();  

    private Singleton (){}

    public static Singleton getInstance() {  
        return INSTANCE;  
    }  
}  
```

有些代码将`INSTANCE`的实例化放在`static`代码块中，笔者认为没有什么差别。

##### 枚举单例(线程安全)

```java
public enum Singleton {
    INSTANCE;

    public void doSomeThing() {……}
}
```

枚举类的构造方法必须是`private`权限修饰，而且JVM保证构造方法只被调用一次，因此枚举类天生单例。枚举单例比普通饿汉式更简洁，并且无偿提供了序列化机制，绝对防止多次实例化（包括多线程安全，因为上述代码反编译之后`INSTANCE`是在`static`块中被实例化额），即使在面对复杂的序列化或者反射攻击甚至克隆的时候。虽然这种方法还没有广泛采用，但是单元素的枚举类型经常成为实现Singleton的最佳方法。

这里插一嘴：如何在其他方式中实现**真·单例模式**呢？即如何防止克隆、反射、序列化带来的多实例问题？

**反射**

上述枚举类反编译后是`public abstract class Singleton extends Enum`，显而易见，`abstract`抽象类自然不能实例化，反射也无能为例。

享有特权的客户端可以借助`AccessibleObject.setAccessible`方法通过反射机制调用私有构造器。而如果需要抵御这种攻击，可以修改构造器，让它在被要求创建第二个实例的时候抛出异常。

比如：
```java
private static boolean flag = true;
private SingleTon() {
    if (flag){
        flag = false;
        ……
    }else {
        throw new RuntimeException("对象已存在");
    }
}
```

**序列化**

枚举的序列化有如下规定：[Serialization of Enum Constants](https://docs.oracle.com/javase/7/docs/platform/serialization/spec/serial-arch.html#6469)

大致意思为：枚举常量的序列化形式仅有它的名称组成，要序列化枚举常量，`ObjectOutputStream`会写入枚举常量名称方法返回的值。要反序列化枚举常量，`ObjectInputStream`从流中读取常量名称;然后通过调用`java.lang.Enum.valueOf`方法获取反序列化的常量。同时，编译器是不允许任何对这种序列化机制的定制的，因此禁用了`writeObject`、`readObject`、`readObjectNoData`、`writeReplace`和`readResolve`等方法。

普通的Java类的反序列化过程中，会通过反射调用类的默认构造函数来初始化对象。所以，即使单例中构造函数是私有的，也会被反射给破坏掉。由于反序列化后的对象是重新new出来的，所以这就破坏了单例。但是，枚举的反序列化并不是通过反射实现的。

普通Java类要想在反序列化中维护并保证Singleton，必须声明所有实例域都是`transient`的，并重写`readResolve`方法：

```java
@Override
private Object readResolve() {
    return getInstance();
}
```

**克隆**

`java.lang.Enum`中的`final`克隆方法确保枚举常量永远不能被克隆

普通Java类通过以下方式即可简单的防止克隆带来的多实例问题：
```java
@Override
protected Object clone() throws CloneNotSupportedException {
    return getInstance();
}
```

#### 懒汉式

懒汉式就是指全局的单例实例在第一次被使用时构建，即延迟加载，或者称为懒加载。

懒汉式在需要的时候才创建对象，有节省内存、节约流量、减轻系统启动时服务器的压力等优点

##### 普通懒汉式

```java
public class Singleton {
    private static Singleton INSTANCE;

    private Singleton(){}

    public static Singleton getInstance() {
        if(INSTANCE == null) {
            INSTANCE = new Singleton();
        }
        return INSTANCE;
    }
}
```

此种实现方式不是线程安全的，多线程情况下仍会有多实例的出现，如下图所示：

![非线程安全懒汉式单例模式线程不安全演示图.png](https://i.loli.net/2019/05/21/5ce3ec58ab94e22634.png)

##### 方法加锁方式(线程安全)

```java
public class Singleton {
    private static Singleton INSTANCE;

    private Singleton(){}

    public static synchronized Singleton getInstance() {
        if(INSTANCE == null) {
            INSTANCE = new Singleton();
        }
        return INSTANCE;
    }
}
```
此种写法能够在多线程中很好的工作，但是每次调用`getInstance`方法时都需要进行同步，造成不必要的同步开销，况且大部分时候我们是用不到同步的。

##### 双重检验加锁方式(线程安全)

```java
public class Singleton {
    //volatile保证INSTANCE变量的可见性和禁止指令重排序
    private volatile static Singleton INSTANCE;

    private Singleton(){}

    public static Singleton getInstance() {
        if(INSTANCE == null) {
            synchronized(Singleton.class) {
                //进入同步代码块后，再检查一次，如果仍是null，才创建实例
                if(INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

这里`getSingleton`方法中对`INSTANCE`进行了两次判空，第一次是为了不必要的同步，那为什么需要第二次判空呢？

这是为了避免出现普通懒汉方式的多实例现象的出现，尽管可能会有多个线程通过第一个第一个判空语句`if(INSTANCE == null)`，但是在某个线程创建完`Singleton`对象时，其他线程继续执行都会被第二个判空语句`if(INSTANCE == null)`挡在外面。这样也就确保了多线程下只有一个实例会被创建。

注意上述所有的说法都是建立在我们假设之上的：
1. 假设一：单实例对象被一个线程创建后立即被其他线程可见
2. 假设二：`INSTANCE != null`代表着单实例对象已经被正常创建完毕。

`INSTANCE`的可见性是由关键字`sychronized`保证的，因为第二次判空是在加锁以后，所以另一个线程一定可以看到这个引用被赋值。那么我们是怎么保证假设二一定成立呢？

有心的读者一定已经发现我们在定义`INSTANCE`变量的时候还给它加上了`volatile`关键字，它的详细介绍我就不再赘述了，本文重点不再这里，简单的说就是保证了变量的可见性（此种方式的可见性并不是由`volatile`关键字保证，而是由`sychronized`关键字保证）并且禁止指令重排序。

因为`INSTANCE = new Singleton();`在Java字节码中会有下面4个步骤：
1. 申请内存空间
2. 初始化默认值（和构造器方法初始化有区别）
3. 执行构造器方法
4. 连接引用和实例

这4个步骤后两个有可能会重排序，1234或者1243都有可能，如若是1243，那么会造成未初始化完全的对象发布。未完全初始化的对象发布，会使程序发生错误，如下图所示：

![未初始化完全的对象发布.png](https://i.loli.net/2019/05/21/5ce3ec7a048d669592.png)

`volatile`关键字禁止指令重排序通过内存屏障来实现：

- 在每个volatile变量写操作的前面插入一个StoreStore屏障。
- 在每个volatile变量写操作的后面插入一个StoreLoad屏障。
- 在每个volatile变量读操作的后面插入一个LoadLoad屏障。
- 在每个volatile变量读操作的后面插入一个LoadStore屏障。

上面所说执行`new`语句时的第四步即是对`volatile`变量的写操作，前后分别插入SS和SL屏障，确保了`执行构造器方法`不会被重排序到`连接引用和实例`后面。

**注意**：`sychronized`虽然可以保证可见性，但那是因为`synchronized`修饰的代码，同一时间只能被同一线程访问。`as-if-serial`语意保证不管怎么重排序，单线程程序的执行结果都不能被改变。编译器和处理器无论如何优化，都必须遵守`as-if-serial`语义。`as-if-serial`语义保证了单线程中，指令重排是有一定的限制的，而只要编译器和处理器都遵守了这个语义，那么就可以认为单线程程序是按照顺序执行的。当然，实际上还是有重排的，只不过我们无须关心这种重排的干扰。所以实际上`sychronized`中还是存在指令重排序的。


##### 静态内部类方式(线程安全)

```java
public class Singleton {
    private static class LazyHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    private Singleton() {}

    public static Singleton getInstance() {
        return LazyHolder.INSTANCE;
    }
}
```

利用静态内部类的特性，外部类`Singleton`加载的时候不会立即加载静态内部类`LazyHolder`，当调用`Singleton.getInstance()`进而执行`return LazyHolder.INSTANCE;`时，由于用到了`LazyHolder`，此静态内部类才会被加载，`INSTANCE`变量才会被实例化。

##### 容器方式

```java
public class SingletonManager {
    private static Map<String, Object> map = new HashMap<>();

    private SingletonManager() {}

    public static void registerService(String key, Object instance) {
        if(!map.containsKey(key)) {
            map.put(key, instance);
        }
    }

    public static ObtainService(String key) {
        return map.get(key);
    }
}
```

使用`SingletonManager`可以将多种单例类统一管理。其中Spring中的单例bean（scope = "singleton"）就是通过Map进行存储管理的。

### 获取access_token的单例模式应用

这里就拿微信公众号开发中从微信接口请求并管理`access_token`举例说明单例模式的简单应用。

有些同学可能没有接触过微信公众号的开发，这里首先说一下`access_token`大致是个什么东西：
1. access_token是公众号的全局唯一接口调用凭据，公众号调用各接口时都需使用access_token。
2. access_token的有效期是7200秒。中控服务器需要根据这个有效时间提前去刷新新access_token。

简单来说就是一个有有效期的验证码，调用公众号接口是需要提供这个全局唯一的`access_token`

下面是笔者当时的实现方式：

```java
public class Singleton {

    //map中包含一个accessToken和缓存的时间戳
    private Map<String,String> map = new HashMap<>();

    private static Singleton singleton = null;

    private Singleton(){}

    //普通懒汉式
    public static Singleton getInstance(){
        if(singleton == null){
            singleton = new Singleton();
        }
        return singleton;
    }


    public Map<String, String> getMap() {
        return map;
    }

    public void setMap(Map<String, String> map) {
        this.map = map;
    }

    public String getAccessToken(String appid, String appsecret){
        String result = null;
        Singleton singleton = Singleton.getInstance();
        Map<String,String> map = singleton.getMap();
        //起始时间
        String time = map.get("time");
        String accessToken = map.get("access_token");

        Long currentDate = new Date().getTime();//单位为 毫秒
        //设置比过期时间少一些---> 7200s过期 设置超过7000s就刷新一次
        if(accessToken != null && time != null && currentDate-Long.parseLong(time) < 7000*1000){
            //从缓存中拿数据为返回结果赋值
            result = accessToken;
        }else{
            //重新获取并覆盖原先的值
            ……
        }

        return result;
    }
}
```
查看更多代码可以移步这里：https://github.com/zhengtianle/wechat-offical-account/blob/master/src/main/java/util/Singleton.java

上面的代码中使用了两种方式实现单例。

其中一个是普通懒汉单例创建方式，主要用来保证map的全局唯一性（当然可以使用static直接声明为全局变量，但是笔者认为所有类都可以直接使用此map不是一个好的设计）。ps: 这里没有涉及多线程，所以不会出现多线程下的多实例

另一个是利用的容器HashMap存储，保证全局唯一`access_token`。至于这里为什么使用map存储`access_token`而不是直接在`Singleton`类中设计一个Sting变量保存，是因为笔者考虑以后若还需要存储别的全局唯一，直接在map中加入即可。如若读者认为笔者考虑欠佳，还请评论指教。


### 参考

- 《Effective Java 中文版》(原书第3版)
- 《Head First设计模式（中文版）》
- 克隆、序列化、反射——单例模式防御心得：https://zhuanlan.zhihu.com/p/28491630
- 为什么我墙裂建议大家使用枚举来实现单例：https://www.hollischuang.com/archives/2498
- java 单例模式中双重检查锁定 volatile 的作用：https://www.zhihu.com/question/56606703