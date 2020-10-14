---
title: '[设计模式]java中的代理模式之女神的饭我包了'
date: 2019-03-19 19:39:05
tags:
- 设计模式
- Java
categories:
- 设计模式
- 结构型模式
---

近些日子冯宝宝身体有些不舒服，她觉得是自己的饮食有问题，但她不知道该如何健康的安排自己的三餐，于是冯宝宝拜托了郑天乐帮她安排自己的三餐，郑天乐便成为了冯宝宝吃饭的“代理对象”！接下来看看这如何利用java中静态代理和动态代理实现吧。

<!-- more -->

### 定义及类图

{% note default %} 
Provide a surrogate or placeholder for another object to control access to it.（为其他对象提供一种代理以控制对这个对象的访问）
{% endnote %}

![](https://upload.wikimedia.org/wikipedia/commons/thumb/7/75/Proxy_pattern_diagram.svg/1024px-Proxy_pattern_diagram.svg.png)

- 冯宝宝: RealSubject
- 郑天乐: Proxy

### 静态代理

- 静态代理在使用时，需要定义接口或者父类，被代理的对象与代理对象一起实现相同的接口或者继承相同的父类。
- 是在静态代理中，编译期就确定了代理对象，即在运行前代理类的.class文件就已经存在了

说了这么多，让我们用代码看看郑天乐是如何给冯宝宝安排健康的三餐的吧🤓

首先创建郑天乐和冯宝宝的公共接口IPerson
```java
public interface IPerson {

    void eatBreakFast();

    void eatLunch();

    void eatDinner();
}
```

接下来让被代理对象冯宝宝实现IPerson接口

```java
public class FengBaoBao implements IPerson {

    @Override
    public void eatBreakFast() {
        System.out.println("我要吃早餐啦！");
    }

    @Override
    public void eatLunch() {
        System.out.println("我要吃午餐啦！");
    }

    @Override
    public void eatDinner() {
        System.out.println("我要吃晚餐啦！");
    }
}
```

然后代理对象郑天乐也要实现IPerson接口

```java
public class ZhengTianLe implements IPerson {

    //代理对象需要拥有被代理对象的实例!!!
    private IPerson target;

    public ZhengTianLe(IPerson target) {
        this.target = target;
    }

    @Override
    public void eatBreakFast() {
        System.out.println("早餐时间到，买一杯紫薯粥一个鸡蛋一个包子");
        target.eatBreakFast();
    }

    @Override
    public void eatLunch() {
        System.out.println("午餐时间到，买一碗担担面外加一根香蕉");
        target.eatLunch();
    }

    @Override
    public void eatDinner() {
        System.out.println("晚餐时间到，买一杯玉米粥一个滕州菜煎饼");
        target.eatDinner();
    }
}
```

然后我们在主方法中执行如下代码

```java
IPerson fbb = new FengBaoBao();
ZhengTianLe ztl = new ZhengTianLe(fbb);
ztl.eatBreakFast();
ztl.eatLunch();
ztl.eatDinner();
```

输出如下

```cosole
早餐时间到，买一杯紫薯粥一个鸡蛋一个包子
我要吃早餐啦！
午餐时间到，买一碗担担面外加一根香蕉
我要吃午餐啦！
晚餐时间到，买一杯玉米粥一个滕州菜煎饼
我要吃晚餐啦！
```

过了几天，郑天乐的舍友靳小鸡因为颓在宿舍懒得去买饭，让郑天乐去帮他买，刚开始郑天乐还不同意，但被靳小鸡打了一顿之后，便也开始帮靳小鸡买饭了🤐，于是靳小鸡也实现IPerson接口：

```java
public class JinXiaoJi implements IPerson {

    @Override
    public void eatBreakFast() {
        System.out.println("lz要吃早餐！");
    }

    @Override
    public void eatLunch() {
        System.out.println("lz要吃午餐！");
    }

    @Override
    public void eatDinner() {
        System.out.println("lz要吃晚餐！");
    }
}
```

于是主方法又加入了下面的代码

```java
IPerson jxj = new JinXiaoJi();
ztl = new ZhengTianLe(jxj);
ztl.eatBreakFast();
ztl.eatLunch();
ztl.eatDinner();
```

后来郑天乐的其他舍友看到靳小鸡的便利，便都用暴力的方式成为了郑天乐的代理对象，郑天乐哭着说：代理模式真好用……

### 动态代理

#### JDK动态代理

- 在运行期间通过反射创建一个实现了一组给定接口的新类(代理类)
- 生成的新类需要被提供一个handler，由handler接管实际的工作
- 动态代理与静态代理相比较，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）在接口方法数量比较多的时候，我们可以进行更灵活的处理，而不需要像静态代理那样每一个方法进行中转。而且动态代理的应用使我们的类职责更加单一，复用性更强

郑天乐给冯宝宝买了一个月的饭，冯宝宝身体好多了，但是脸也慢慢肥起来了，于是冯宝宝决定早餐和午餐都只吃一杯粥和一个鸡蛋，并且不吃晚餐来减掉自己脸上的肉。这样，郑天乐就需要用jdk的动态代理来改变现有的设计了！

{% note danger %} 
- 代理类所在包:java.lang.reflect.Proxy
- JDK实现代理只需要使用newProxyInstance方法
`static Object newProxyInstance(ClassLoader loader, Class [] interfaces, InvocationHandler handler)`
1. ClassLoader loader: 指定当前目标对象使用类加载器,用null表示默认类加载器
2. Class [] interfaces: 需要实现的接口数组
3. InvocationHandler handler: 执行目标对象的方法时,会触发调用处理器的方法,从而把当前执行目标对象的方法作为参数传入
- java.lang.reflect.InvocationHandler: 调用处理器接口，里面有一个需要被实现类自定义实现的`Object invoke(Object proxy, Method method, Object[] args)`方法，用于集中处理在动态代理类对象上的方法调用，通常在该方法中实现对委托类的代理访问。
1. Object proxy: 代理类实例
2. Method method: 被调用的方法对象
3. Object[] args: 调用参数 
{% endnote %}

**注**: FengBaoBao类和IPerson接口不变，去掉ZhengTianLe类

创建ProxyFactory类实现InvocationHandler,重写invoke方法

```java
public class ProxyFactory implements InvocationHandler {
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }

    //获得代理对象
    public Object getProxyeeInstance() {
        return Proxy.newProxyInstance(
            target.getClass().getClassLoader(), 
            target.getClass().getInterfaces(), 
            this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object returnData = null;
        //早餐或者午餐
        if(method.getName().equals("eatBreakFast") || method.getName().equals("eatLunch")) {
            System.out.println("吃饭时间到，买一杯粥和一个鸡蛋");
            //target.method(args);
            returnData = method.invoke(target, args);
        } else {
            //晚餐不用买
            System.out.println("冯宝宝要减小肥脸，不用买晚饭");
        }
        return returnData;
    }
}
```

现在的主方法就可以这么写

```java
IPerson fbb = new FengBaoBao();
ProxyFactory proxyFactory = new ProxyFactory(fbb);
//创建代理对象
IPerson ztl = (IPerson) proxyFactory.getProxyeeInstance();
ztl.eatBreakFast();
ztl.eatLunch();
ztl.eatDinner();
```

输出如下

```console
吃饭时间到，买一杯粥和一个鸡蛋
我要吃早餐啦！
吃饭时间到，买一杯粥和一个鸡蛋
我要吃午餐啦！
冯宝宝要减小肥脸，不用买晚饭
```

#### Cglib动态代理

JDK动态代理确实比静态代理好处多多，但是还是脱离不了被代理的类需要实现接口的桎梏，但是有时候我们的目标对象就只是一个单独的对象，并没有实现任何接口怎么办呢？

为了弥补这个缺陷，我们可以引入Cglib库，使用它可以在系统运行期间动态的为目标对象生成相应的扩展子类，这种代理方式也就叫做Cglib代理

- 底层使用字节码处理框架ASM处理生成新的子类
- 注意代理的类不可谓final，目标对象的方法也不可为static/final

冯宝宝通过郑天乐安排的健康饮食，又恢复了原先傲人的身材，现在身体也棒棒的。但是郑天乐还是想每顿饭给冯宝宝送一杯粥，利用送粥的时间抱她一下。

FengBaoBao类不实现任何接口

```java
public class FengBaoBao {

    public void eat() {
        System.out.println("我要吃饭啦！");
    }
}
```

要对FengBaoBao类进行扩展，首先需要实现一个`net.sf.cglib.proxy.Callback`，不过更多的时候我们直接使用`net.sf.cglib.proxy.MethodInterceptor`接口(MethodInterceptor扩展了Callback接口)

```java
public class ProxyFactory implements MethodInterceptor {
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }

    //获得代理对象
    public Object getProxyInstance(){
        //1.工具类
        Enhancer en = new Enhancer();
       //2.设置父类
        en.setSuperclass(target.getClass());
        //3.设置回调函数
        en.setCallback(this);
        //4.创建子类(代理对象)
        return en.create();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("送给你一杯粥，抱抱");
        //执行目标对象的方法
        Object returnValue = method.invoke(target, args);
        return returnValue;
    }
}
```

现在的主方法这么写

```java
//目标对象
FengBaoBao fbb = new FengBaoBao();
//代理对象
ProxyFactory proxyFactory = new ProxyFactory(fbb);
FengBaoBao ztl = (FengBaoBao) proxyFactory.getProxyInstance();
ztl.eat();
```

输出如下

```console
送给你一杯粥，抱抱
我要吃饭啦！
```

最后，郑天乐和冯宝宝过上了幸福美好的生活……