---
title: Spring注解驱动开发-组件的注册与扫描
date: 2018-08-08 19:35:35
tags: 
- Spring
- Java
categories: 
- Spring
- Spring注解驱动开发
---

这个时代中,Spring就像是Java的衣服一样,作为一个Javaer,我总不能光着身子跑吧😆.恰巧在bilibili发现这门"尚硅谷Java视频教程_Spring注解驱动开发视频教程",便借此机会写入博客以助理解.本文将主要介绍利用Spring注解进行组件注册与扫描的四种方式:包扫描,@Bean注解,@Import注解,FactoryBean的使用...

<!-- more -->


## Spring注解给容器中注册组件的四种方式:

{% note info no-icon %}
1. 包扫描+组件标注注解(@Controller/@Service/@Repository/@Component)[适用于自定义类,方便自行添加注解]
2. @Bean注解[适用于导入第三方包内的组件]
3. @Import[适用于快速给容器中导入组件]
3.1 @Import({Classes}),Classes可为普通类,实现ImportSelector接口类和实现ImportBeanDefinitionRegistrar接口类:容器会自动注册这些组件,id默认为全类名
3.2 ImportSelector:返回需要导入组件的全类名数组
3.3 ImportBeanDefinitionRegistrar:手动注册Bean到容器中
4. 使用Spring提供的FactoryBean
4.1 applicationContext.getBean(String name);默认获取到的是FactoryBean调用getObject创建的对象
4.2 若要获取FactoryBean本身,需要给getBean参数前面加上&,如getBean(&colorFactoryBean)
{% endnote %}

**本文使用的环境是Ubuntu16.04LTS+Intellij idea2018.1.2+jdk1.8+maven3.53
下面代码均在maven导入spring-context以及junit前提下运行测试成功
pom.xml中依赖:**
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.3.18.RELEASE</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>
```

## 包扫描+组件标注注解

用SSM框架进行开发时可能会用到大量的xml进行配置整合,但是了解过Spring Boot以及Spring Cloud的码友就会发现Spring注解用的越来越多,这其实也是我们选择去多多使用注解的原因之一.简单的说Spring注解开发就是把xml配置文件替换为Java配置类,不废话,接下来我们边码边说.

### 配置类注解:@Configuration
类注解,告诉Spring此类为配置类(相当于配置文件)
```java
@Configuration
public class MainConfig {}
```
### 包扫描注解:@ComponentScan

配置包扫描之前我们先在工程中添加几个可以可以被自动扫描和装配的类,主要用到的注解就是@Component(本质上@Controller,@Service,@Repository都是@Component):
```java
@Service
public class BookService {}
```
```java
@Repository
public class BookDao {}
```
```java
@Controller
public class BookController {}
```

- **value:指定要扫描的包**
```java
@Configuration
//扫描com.atguigu包
@ComponentScan(value = "com.atguigu")
public class MainConfig {}
```
- **excludeFilters = Filter[]:指定扫描的时候按照某种规则不包含某些组件**
```java
//按照注解去除com.atguigu包内含有@Controller,@Service注解的类
@ComponentScan(value = "com.atguigu",excludeFilters = {
        @ComponentScan.Filter(type=FilterType.ANNOTATION,
        classes = {Controller.class,Service.class})
        })
```
- **includeFilters = Filter[]:指定扫描的时候按照某种规则只包含某些组件,前提是先禁用默认规则:useDefaultFilters = false**
```java
//按照注解只扫描装配com.aiguigu包内的含有@Controller,@Service注解的类
@ComponentScan(value = "com.atguigu",includeFilters = {
        @ComponentScan.Filter(type=FilterType.ANNOTATION,
        classes = {Controller.class,Service.class})
},useDefaultFilters = false)
```
- **@ComponentScan在java8里面是可重复(@Repeatable)组件,不是java8可以用@ComponentScans(value = {@ComponentScan[]})**

1. 在java8里面一个配置类可以有多个@ComponentScan注解,例如可以把前面介绍的两个过滤规则同时写在一个配置类上(当然,这个例子没什么意义,因为先去除了含有@Controller,@Service注解的类又只扫描装配含有@Controller,@Service注解的类,相当于不装配任何类😂)

2. java8以下版本也可以通过@ComponentScans注解配置多个扫描:
```java
@ComponentScans({@ComponentScan,@ComponentScan})
```

{% note danger no-icon %}
其中@Filter的type可为五中类型:
- [x] **FilterType.ANNOTATION**:按照注解进行过滤
- [x] **FilterType.ASSIGNABLE_TYPE**:按照给定的类型过滤
- [ ] **FilterType.ASPECTJ**:使用ASPECTJ表达式过滤
- [ ] **FilterType.REGEX**:使用正则表达式过滤
- [x] **FilterType.CUSTOME**:使用自定义规则过滤
{% endnote %}

- **FilterType.ANNOTATION:按照注解进行过滤**
前面的代码演示中已经使用了FilterType.ANNOTATION,简单来说就是根据注解进行过滤
- **FilterType.ASSIGNABLE_TYPE:按照给定的类型过滤**
我们可以在之前的过滤规则中再加一条使用FilterType.ASSIGNABLE_TYPE的过滤规则:
```java
//满足第一条过滤规则的前提下,扫描装配BookService类(其实这个例子也没什么用😂)
@ComponentScan(value = "com.atguigu",includeFilters = {
        @ComponentScan.Filter(type=FilterType.ANNOTATION,
        classes = {Controller.class,Service.class}),
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,
        classes = {BookService.class})
},useDefaultFilters = false)
```
- **FilterType.CUSTOME:使用自定义规则(自定义规则是一个实现TypeFilter的类)过滤**

1. 首先我们创建一个实现TypeFilter接口的自定义规则类:
```java
public class MyTypeFilter implements TypeFilter {

    /**
     * @param metadataReader    读取到的当前正在扫描的类的信息
     * @param metadataReaderFactory 探索获取到其他任何类的信息
     * @return true代表允许扫描装配,false代表不允许
     */
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
            throws IOException {

        //可以获取当前类注解信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        //可以获取当前正在扫描的类的类信息(比如类型,实现了什么接口等)
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        //可以获取当前类资源(类的路径)
        Resource resource = metadataReader.getResource();
        //简单的使用类名过滤一下
        String className = classMetadata.getClassName();
        System.out.println("当前类的类名  " + className);
        if(className.contains("er"))
            return true;
        return false;
    }
}
```
2. 在我们的MainConfig类上的@ComponentScan中使用这个自定义过滤规则:
```java
@ComponentScan(value = "com.atguigu",includeFilters = {
        @ComponentScan.Filter(type=FilterType.ANNOTATION,
        classes = {Controller.class,Service.class}),
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,
        classes = {BookService.class}),
        @ComponentScan.Filter(type = FilterType.CUSTOM,
        classes = {MyTypeFilter.class})
},useDefaultFilters = false)
```

## @Bean注解

{% note info no-icon %}
@Bean主要被用在方法上,显示声明要生成的类
{% endnote %}

首先我们新建一个配置类,利用@Bean注解在类里显示声明一个bean:
```java
public class MainConfig {
    /**
     * 给容器注册一个bean,类型为返回值的类型,id如果不指定则方法名为默认id
     * 此处注册的bean的id为 fengbao
     */
    @Bean("fengbao")
    public Person person(){
        return new Person("冯宝宝",19);
    }
}
```
其中Person类是一个有且只有name(String)和age(Integer)属性的简单类

{% note primary no-icon %}
- **单实例懒加载Bean** :默认不开启;@Lazy启用懒加载,Spring容器启动的时候不会创建实例bean,当调用getBean的时候会创建实例bean并放到容器中
- **单实例非懒加载Bean** :**默认开启;** 即为@Scope("singleton"),Spring容器启动的时候会调用方法创建实例bean并放到容器中,之后每次getBean获取都是直接从容器(map.get())中获取
- **非单实例Bean** :默认不开启;即为@Scope("prototype"),Spring容器启动的时候不会创建实例bean,每次获取的时候才会调用方法创建对象
{% endnote %}

### @Scope注解:设置组件作用域

{% note danger no-icon %}
其中@Filter的value可为四种:
- [x] **SCOPE_PROTOTYPE(prototype)** :多实例
- [x] **SCOPE_SINGLETON(singleton)** :单实例
- [ ] **SCOPE_REQUEST(request)** :web中的同一次请求创建一个实例
- [ ] **SCOPE_SESSION(session)** :web中的同一个session创建一个实例
{% endnote %}

```java
// 单实例非懒加载Bean
@Bean("singletonBean")
public Person singletonPerson() {
    return new Person("singleton冯宝宝", 19);
}

//非单实例Bean
@Scope("prototype")
@Bean("prototypeBean")
public Person prototypePerson() {
    return new Person("prototype冯宝宝", 19);
}
```

### @Lazy注解:bean懒加载

```java
//单实例懒加载Bean
@Lazy
@Bean("lazyBean")
public Person lazyPerson() {
    return new Person("lazy冯宝宝", 19);
}
```

说到这,我们可能会觉得单实例懒加载与非单实例完全相同,其实它们在底层也是有些区别的,这里贴一个博文链接:[非单实例与单实例懒加载的区别](http://blog.zdydoit.com/blogs/2018/08/spring-ioc-lazy-scope-source)

## @Conditional注解
边码边说:
```java
public class MainConfig2{

    ...

    /**
     * @Conditional({Condition}):按照一定的条件进行判断,满足条件给容器注册bean
     *
     * 现在我们设置我们的条件判断:
     * 如果当前系统是linux,则在容器中注册linus
     * 如果是windows系统,则在容器中注册bill
     */
    @Conditional(WindowsCondition.class)
    @Bean("bill")
    public Person person01(){
        return new Person("Bill Gates",62);
    }

    @Conditional(LinuxCondition.class)
    @Bean("linus")
    public Person person03(){
        return new Person("linus",48);
    }
}
```
LinuxCondition.class:
```java
public class LinuxCondition implements Condition {

    /**
     * @param context   判断条件能使用的上下文(环境)
     * @param metadata  当前标注了@Conditional注解的注释信息
     * @return true代表注册linus到容器中
     */
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //判断是否是linux系统
        //1.可以获取到ioc的使用的beanFacytory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        //2.可以获取到类加载器
        ClassLoader classLoader = context.getClassLoader();
        //3.可以获取到当前环境信息(环境变量,虚拟机设置变量等等)
        Environment environment = context.getEnvironment();
        //4.可以获取到bean定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();

        //简单的利用当前系统信息判断
        String property = environment.getProperty("os.name");
        //可以判断bean的注册情况,也可以给容器中注册bean:registry.registerBeanDefinition();
        boolean definition = registry.containsBeanDefinition("person");
        
        if(property.contains("Linux")){
            return true;
        }
        return false;
    }
}
```
WindowsCondition.class:
```java
public class WindowsCondition implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        if(property.contains("Windows"))
            return true;
        return false;
    }
}
```

*@Conditional注解也可以是类注解,作为类注解,它表示:类中组件统一设置满足当前condition条件,此类中的所有bean才能生效*

## @Import注解

### 直接导入
直接在类注解@Import中的value值中加入要导入的class:
```java
//@Import导入组件,id默认是组件的全类名,比如Color.class的id是com.atguigu.bean.Color
@Import({Color.class,Red.class})
public class MainConfig2{...}
```

### ImportSelector选择器导入
ImportSelector接口实现类:
```java
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {
    /**
     * @param importingClassMetadata 当前标注@Import注解的类的所有注解信息
     * @return 要导入到容器中的组件全类名数组
     */
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        //可以通过importingClassMetadata得到所有注解的属性以及给个类的信息
        //这里简单返回一个Blue类的全类名作为演示
        return new String[]{"com.atguigu.bean.Blue",};
    }
}
```
配置类中使用这个选择器:
```java
@Import({Color.class,Red.class,MyImportSelector.class})
public class MainConfig2{...}
```

### ImportBeanDefinitionRegistrar手动导入

ImportBeanDefinitionRegistrar接口实现类:
```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    /**
     * @param importingClassMetadata 当前类的注解信息
     * @param registry BeanDefinitionRegistry的注册类
     *                 所有需要添加到容器中的bean都可以使用BeanDefinitionRegistry.registerBeanDefinition手工注册进来
     */
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        //判断Spring的IoC容器中是否有Red类和Blue类
        boolean definition = registry.containsBeanDefinition("com.atguigu.bean.Red");
        boolean definition1 = registry.containsBeanDefinition("com.atguigu.bean.Blue");

        //如果红色蓝色同时存在则手动注册RainBow类到Spring容器中
        if(definition && definition1){
            //制定Bean的类型(Bean的类型,Bean作用域等等)
            RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
            //注册一个Bean,指定bean名字
            registry.registerBeanDefinition("rainBow",beanDefinition);
        }
    }
}
```
配置类中使用这个手动注册器:
```java
@Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})
public class MainConfig2{...}
```

## FactoryBean

{% note info no-icon %}
- 普通的Bean:直接导入容器中,容器会调用Bean类的无参构造器创建对象,注册在容器中.
- Spring提供的FactoryBean(工厂Bean)是一个接口:
    1. 容器会调用接口中的getObject()方法,将返回的对象放入容器中.
    2. 可以通过getObjectType()来控制返回对象类型
    3. 可以通过isSingleton()来设置是否单例
{% endnote %}

FactoryBean<T>接口实现类:
```java
public class ColorFactoryBean implements FactoryBean<Color> {
    //返回一个Color对象,这个对象会添加到容器中
    public Color getObject() throws Exception {
        System.out.println("ColorFactoryBean...getObject...");
        return new Color();
    }

    public Class<?> getObjectType() {
        return Color.class;
    }

    /**
     * 是否单例
     * @return true表示单实例:容器中只保存一份
     *         false表示多实例:每次获取都会调用工厂Bean的getObject()方法创建对象
     */
    public boolean isSingleton() {
        return false;
    }
}
```

配置类中使用FactoryBean:
```java
public class MainConfig2{
    ...
    @Bean
    public ColorFactoryBean colorFactoryBean(){
        return new ColorFactoryBean();
    }
}
```

**这里需要多说一句的是使用getBean()默认获取到的是FactoryBean调用getObject()创建的对象,想要获取工厂Bean对象本身,需要使用BeanFactory中定义的`String FACTORY_BEAN_PREFIX = "&";`**

我们可以建立一个测试类来测试一下:
```java
public class IOCTest{

    @Test
    public void factoryBeanTest(){
        //获取配置上下文
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        //打印容器中的bean的name
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for(String name : definitionNames){
            System.out.println(name);
        }

        //测试ColorFactoryBean类中的isSingleton()方法
        Object bean1 = applicationContext.getBean("colorFactoryBean");
        Object bean2 = applicationContext.getBean("colorFactoryBean");
        System.out.println("colorFactoryBean的类型: "+bean1.getClass());
        System.out.println(bean1 == bean2?"两次获取的ColorFactoryBean一样":"两次获取的ColorFactoryBean不一样");

        //打印工厂Bean创建的对象类型
        Object bean3 = applicationContext.getBean("colorFactoryBean");
        System.out.println("colorFactoryBean的类型: "+bean3.getClass());
        
        //打印工厂bean的类型
        Object bean3 = applicationContext.getBean("&colorFactoryBean");
        System.out.println("&colorFactoryBean的类型: "+bean3.getClass());

    }
}
```

## 附录
bilibili视频链接: https://www.bilibili.com/video/av20967380


