---
title: '[设计模式]Facade（外观）模式有什么用？'
date: 2019-08-03 22:26:25
tags:
- 设计模式
categories:
- 设计模式
- 结构型模式
---

实习的时候看公司软件项目设计文档，发现里面多处提及了Facade（外观）设计模式，一般对于 Facade 的描述是这样子的：“服务内部划分为多个相对独立的组件，使用 Facade 设计模式，将这些组件的接口封装成服务接口，对外部服务提供同意的设备管理接口”。了解 Facade 模式的同学肯定知道这段描述的很清楚，但是 Facade 的功能仅仅是这样子吗？或者说这句话所隐含的意义是我们理解的那样子吗？像笔者这种对设计模式一知半解的人，可能就有点迷了。所以，笔者写了一篇 Facade 模式的总结，学习分享一下。

<!-- more -->

## 定义

{% note default %} 
它为子系统中的一组接口提供一个统一的高层接口，使得子系统更容易使用。
PS：外观模式也称门面模式
{% endnote %}

![Facade设计模式.png](https://i.loli.net/2019/08/04/dXIM6E3QmlfnW8N.png)

- **Facade（外观角色）**：客户端可以调用此角色中的方法，此角色知晓相关（一个或者多个）子系统的功能和责任；正常情况下，此角色会将所有从客户端发来的请求委派到相应的子系统中。
- **SubSystem（子系统角色）**：软件系统可以有一个或者多个子系统。每一个子系统可以不是一个单独的类，而是一个类的集合（上图中的子系统由 ModuleA、ModuleB、ModuleC 三个类组合而成）。每个子系统都可以被客户端（client）直接调用，或者被外观角色调用。注意，子系统并不知道外观角色的存在，对于子系统而言，外观角色仅仅是另外一个客户端。

图中的 Facade 类可以这么实现：
```java
public class Facade {
    public void wrapOperation() {
        Module a = new ModuleA();
        Module b = new ModuleB();
        Module c = new ModuleC();

        //示意逻辑
        a.operation();
        b.operation();
        c.operation();
    }
}
```

客户端使用系统功能时：
```java
public class Client {
    public static void main(String[] args) {
        Facade facade = new Facade();
        facade.wrapOperation();
    }
}
```

这里我们可以看到，有了 Facade 类后，客户端不需要亲自调用子系统中的三个模块，也不需要知道系统内部的实现细节，甚至都不需要知道 A、B、C 模块的存在。这样的实现，使得子系统的三个模块与客户端之间解耦，客户端也更容易使用子系统。

## 遥控器的例子

单单看上面的定义可能不好理解，也不够深刻，其实我们可以联想到我们生活中使用的遥控器，个人觉得是一个非常经典的例子，下面我们可以用代码简单的模拟一下电视遥控器。

以下把遥控器简单的理解成只有三个子系统：`电源系统`、`声音系统`、`频道系统`

```java
public class TvRemoteController {
    private PowerSystem power = new PowerSystem();
    private VoiceSystem voice = new VoiceSystem();
    private ChannelSystem channel = new ChannelSystem();

    public void turnOn() {
        power.turnOn();
        channel.turnOn();
        voice.turnOn();
    }

    public void turnOff() {
        voice.turnOff();
        channel.turnOff();
        power.turnOff();
    }

    public void turnUp() {
        voice.turnUp();
    }

    public void turnDown() {
        voice.turnDown();
    }

    public void previousChannel() {
        channel.previous();
    }

    public void nextChannel() {
        channel.next();
    }
}
```

看这个遥控器类，它对`电源系统`、`声音系统`、`频道系统`做了封装，当我们需要打开电视机的时候，不再需要对每个子系统按照某种顺序操作某些功能了，而是直接点击遥控器上的按钮`turnOn`即可！并且我们还可以发现，如果我们只操作遥控器的话，我们是不知道有这三个子系统的存在的，也不知道这些子系统有哪些功能，这也就避免了我们因为对各个系统的不熟悉造成某些故障，毕竟不是每一个人都是工程师，都懂得每个子系统该如何取运行才能完成最终的功能。

我们可以通过遥控器的例子总结一下外观模式的优点：
- **松散耦合**：松散了客户端与子系统的耦合关系，子系统内部模块更易扩展和维护 
- **简单易用**：客户端不需要与子系统内部复杂的内部模块交互
- **合理划分访问层次**：把需要暴露给外部的功能集中到外观角色中，这样既方便客户使用，也很好的隐藏了内部的细节。

## Java中的外观模式

Java中有很多应用到外观模式的案例，我们挑两个比较典型且常见的说一说。

### Tomcat与外观模式

使用过 Servlet 开发的人都知道 HttpServlet 抽象类，我们一般需要重写它的 doGet 或 doPost 方法。而这两个方法的参数都是 `HttpServletRequest reuest，HttpServletResponse response`。但是我们可以看一下 Tomcat 实现的 Servlet 代码的调试图：
![Tomcat调试过程.png](https://i.loli.net/2019/08/04/vXkyNIwnQa8pCqB.png)

从方法调用栈中我们可以看出，`HttpServletRequest`参数位置本来是`Request`，`HttpServletResponse`参数位置本来是`Response`，这是怎么回事呢？我们看一下红色方框标注的 StandardWrapperValue 类中的 invoke 方法第225行：
```java
filterChain.doFilter (request.getRequest(), response.getResponse());
```
继续进行看 getRequest() 与 getResponse()：
```java
public HttpServletRequest getRequest() {
    if (facade == null) {
        facade = new RequestFacade(this);
    }
    return facade;
}

public HttpServletResponse getResponse() {
    if (facade == null) {
        facade = new ResponseFacade(this);
    }
    return (facade);
}
```
发现原来我们使用的`HttpServletRequest`和`HttpServletResponse`是`Request`和`Response`类的外观类，真相大白了！

那么问题来了，这么做有什么好处呢？

因为 `Request`对象中的很多方法都是内部组件之间交互时使用的，例如 setComet、setRequestedSessionId 等方法。这些方法并不应该对外公开，但是又必须设置为 public ，因为还需要和内部组件进行交互。最好的解决办法就是通过一个 Facade 类，将与内部组件之间交互使用的方法屏蔽掉，只提供给外部程序感兴趣的方法。

### SLF4J与外观模式

SLF4J 的全称：Simple LOgging Facade for Java，即 java 的简单日志门面。

为什么说它使用了外观模式，显然，从名字就可以看出来了……开个玩笑，那是因为 SLF4J 不提供日志的具体实现，只作为一个门面接口。下面我们新建一个项目看一下：

导入slf4j包：
```xml
<dependency>
    <group>org.slf4j</group>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.26</version>
</dependency>
```
创建一个 App 类用于测试：
```java
public class App {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(App.class);
        loggger.info("SLF4J!");
        System.out.println("Hello World!");
    }
}
```
运行结果：
```console
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation(NOP) logger implementationSLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for futher details.
Hello World!
```
我们去 jar 中是找不到`org.slf4j.impl.StaticLoggerBinder`这个类的，这也验证了我们上面所说的 SLF4J 不提供具体实现，而这个类你可以在 logback 等具体的日志包中找到（其实这也是 SLF4J 可以自动识别实际上的 logger 日志的原理）

通过SLF4J，开发者就不需要关注具体的实现，而只需要使用 SLF4J 提供的 API 即可。

## 后话

其实外观模式就是“迪米特法则”（一个对象应当对其他对象有尽可能少的了解）的一个实现，通过引入一个外观类，使得客户端与子系统内部的对象的相互作用被外观对象所取代。外观类充当了客户端与子系统类之间的“第三者”，降低了客户端与子系统之间的耦合度，并在一定程度上屏蔽了某些底层方法（为什么说是一定程度上？因为如果客户端想要绕过外观模式直接使用子系统，我们也拦不住……）

但是它也确实违背了“开闭原则”（软件中的对象（类、模块、函数等等）应该对与扩展是开放的，但是对于修改是封闭的），因为当增加或者移除子系统时可能需要修改外观类或者客户端的源代码。如果我们试图解决这个问题，可以引入抽象外观类，客户端针对抽象外观类进行编程。对于新的业务，不修改原有外观类，而对应增加一个新的具体外观类，由新的外观类来关联新的子系统对象，同事通过修改配置文件来达到不修改源代码并且更换外观类的目的。但是我们真的要这么做吗？笔者觉得不一定，设计模式只是一种前任总结的经验，可以参考，重要的是遵循业务模型，开发者首先要做的是对核心概念、业务现状与发展方向的把握，若业务的本质内容已被推翻，又何必往新业务上强行扭转呢？

## 参考

[java设计模式－门面模式(外观模式 Facade)](https://www.jianshu.com/p/b52db22657f3)