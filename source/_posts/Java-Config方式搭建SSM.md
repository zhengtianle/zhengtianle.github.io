---
title: Java Config方式搭建SSM
date: 2018-11-29 09:43:16
tags:
- Spring
- Java
- Spring MVC
- Mybatis
categories: 
- SSM
- JavaConfig搭建框架
---

终于到第十二周了，软件工程与实践课设也到了开始做项目的时刻，上学期数据库课设自己用纯servlet做的App(现在再看其中的代码，不忍直视😑)，但这学期新接触了Spring，SpringMVC和Mybatis，于是选用较为流行的SSM框架编写这次课设的后端代码。谷歌百度到处都充斥着大家用xml配置文件搭建的ssm框架，copy起来一点儿也不费劲，但是作为“世一大”的学子这样可不能为天下储人才，为国家图富强😎(其实是业界普遍开始推荐JavaConfig方式，而且这种配置方式可读性和可维护性更高)，于是本人选择零xml配置搭建SSM框架，本文记录此简易SSM框架搭建过程。

<!-- more -->

{% note info no-icon %}

**本机环境：**
- Manjaro 18.0.0 Illyria(Arch系linux)
- IntelliJ IDEA 2018.2.5(Ultimate Edition)
- java version "1.8.0_191"

{% endnote %}

## 创建JavaWeb项目
本文重点不在这里，详细步骤还是自行百度吧(穷人没有那么多流量上传图片🤒)
[idea创建maven web项目](http://buhuibaidu.me/?s=idea%E5%88%9B%E5%BB%BAmaven%20web%E9%A1%B9%E7%9B%AE)

## 项目目录结构
```
src  
│
└───main
│   │
│   └───java
│   │   │
│   │   └───config
│   │   │       Config.java
│   │   │       RootConfig.java
│   │   │       WebAppInitializer.java
│   │   │       WebConfig.java
│   │   └───controller
│   │   │       HomeController.java
│   │   └───dao
│   │   └───mapper
│   │   └───pojo
│   │   └───service
│   │   └───utils
│   └───resources
│   │   │       db.properities
│   │   │       generatorConfig.xml
│   │   │       log4j2.xml
│   │   │       mybatis-config.xml
│   │   └───mapperXml
│   │   │
│   └───webapp
│   │   │       index.jsp
│   │   └───WEB-INF
│   │       │    web.xml
│   │       └───view
│   │               home.jsp
│   │   
└───test
    │
    └───controller
    │       HomeControllerTest.java
    └───mapper  
            PersonMapperTest.java
```


## Maven导入依赖pom.xml

### properties
```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>

    <!-- spring版本号 -->
    <!--<spring.version>4.3.20.RELEASE</spring.version>-->
    <spring.version>5.1.3.RELEASE</spring.version>
    <!-- mybatis版本号 -->
    <mybatis.version>3.4.6</mybatis.version>
    <!-- mysql版本号 -->
    <mysql.version>8.0.13</mysql.version>
    <!--log4j 版本号-->
    <log4j.version>2.11.1</log4j.version>
</properties>
```
**注**：本文依赖均使用的是[maven公共仓库](https://mvnrepository.com/)最新版，如果需要其他版本更改`<version>`即可

### dependencies

```xml

<dependencies>
    <!--mybatis 3.4.6-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>${mybatis.version}</version>
    </dependency>

    <!-- mybatis-generator-core 1.3.7：逆向工程-->
    <dependency>
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-core</artifactId>
        <version>1.3.7</version>
    </dependency>

    <!-- mybatis-spring整合:1.3.2 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>1.3.2</version>
    </dependency>

    <!-- mysql-connector-java 8.0.13 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
    </dependency>

    <!-- druid1.1.12：
    数据库连接池，自带监控页面，实时监控应用的连接池情况 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.12</version>
    </dependency>

    <!-- lombok 1.18.4 -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.4</version>
        <scope>provided</scope>
    </dependency>

    <!-- javaee-api 8.0 -->
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-api</artifactId>
        <version>7.0</version>
        <scope>provided</scope>
    </dependency>

    <!-- servlet-api 4.0.1 -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
        <scope>provided</scope>
    </dependency>


    <!-- log4j-core 2.11.1-->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>${log4j.version}</version>
    </dependency>

    <!-- log4j-api 2.11.1 -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>${log4j.version}</version>
    </dependency>

    <!-- log4j-slf4j-impl: 2.11.1 -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>${log4j.version}</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.25</version>
    </dependency>

    <!--Spring-->
    <!--spring的核心工具包-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <!-- spring-beans：
    spring ioc基础实现，包含访问配置文件、创建和管理bean等 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <!-- spring-context：
        在基础ioc功能上提供扩展服务，此外还提供许多企业级服务的支持，如邮件服务、任务调度、
        JNDI定位，EJB集成、远程访问、缓存以及多种视图层框架的支持-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <!-- spring-jdbc：
        对JDBC的简单封装-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <!-- spring-web：
        包含web应用开发时用到spring框架所需要的核心类
        包括自动载入WebApplicationContext特性的类-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <!-- spring-webmvc：
        包含spring mvc框架相关的所有类。
        包含国际化、标签、Theme、视图展现的FreeMarker、JasperReports、 Tiles、Velocity、XSLT相关类-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <!-- spring-aop：
        spring的面向切面编程实现-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <!-- spring-tx：spring事务 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <!-- spring-test：
        对junit等测试框架的简单封装-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
        <scope>test</scope>
    </dependency>


    <!--junit单元测试-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>RELEASE</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

### plugins

在`<plugins>`里面增加`mybatis-generator-maven-plugin`插件：
```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.7</version>
    <configuration>
        <!--是否将生成过程输出至控制台-->
        <verbose>true</verbose>
        <!--是否覆盖：是-->
        <overwrite>false</overwrite>
    </configuration>
</plugin>
```
**注**：刚创建的项目`<plugins>`还会有一个父节点`<pluginManagement>`，如果你不知道你用它可以干什么，就把它删掉，详情右转 [Maven中plugins和pluginManagement](https://www.jianshu.com/p/49acf1246eff)

## 搭建 Spring MVC

### 配置 DispatcherServlet

#### WebAppInitializer.java

> &nbsp;&nbsp;&nbsp;&nbsp;`DispatcherServlet`是Spring MVC的核心。在这里请求会第一次接触到框架，它主要负责将请求路由到其他的组件之中。
> &nbsp;&nbsp;&nbsp;&nbsp;按照传统的方式,像`DispatcherServlet`这样的Servlet会配置在`web.xml`文件中，这个文件会放到应用的`WAR`包里面。当然，这是配置`DispatcherServlet`的方法之一。但是，借助于`Servlet3`规范和`Spring3.1`的功能增强，这种方式已经不是唯一的方案了，这也不是我们本文所使用的配置方法。
> &nbsp;&nbsp;&nbsp;&nbsp;我们会使用`java`将`DispatcherServlet`配置在`Servlet`容器中，而不会再使用`web.xml`文件。如下程序清单展示了所需的java类。

```java
package config;


import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author ZhengTianle
 * Description:
 * 配置DispatcherServlet
 */
public class WebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    private static final Logger LOG = LoggerFactory.getLogger(WebAppInitializer.class);

    @Override
    protected Class<?>[] getRootConfigClasses() {
        LOG.info("------root配置类初始化------");
        return new Class<?>[] { RootConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        LOG.info("------web配置类初始化------");
        return new Class<?>[] { WebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        LOG.info("------映射根路径初始化------");
        return new String[]{ "/" };//将DispatcherServlet映射到"/"
    }
}
```

{% note info no-icon %}

- Spring Web应用中通常存在两个应用上下文，一个是`DispatcherServlet`创建的应用上下文，一个是`ContextLoaderListener`创建的应用上下文。
- `AbstractAnnotationConfigDispatcherServletInitializer`会同时创建`DispatcherServlet`和`ContextLoaderListener`。
- `getServletConfigClasses()`方法返回的带有`@Configuration`注解的类会用来定义`DispatcherServlet`应用上下文中的bean：包含Web组件的bean，如控制器、试图解析器以及处理器映射等。
- `getRootConfigClasses`方法返回的带有`@Configuration`注解的类将会用来配置`ContextLoaderListener`创建的应用上下文中的bean：通常是驱动应用后端的中间层和数据库组件。

{% endnote %}

### 启用 Spring MVC

> &nbsp;&nbsp;&nbsp;&nbsp;我们有多种方法来配置`DispatcherServlet`，与之类似，启用Spring MVC组件的方法也不只一种。从前，Spring是使用`XML`进行配置的，可以使用`<mvc:annotation-driven>`启用注解驱动的Spring MVC。
> &nbsp;&nbsp;&nbsp;&nbsp; 不过现在我们会让Spring MVC的搭建过程尽可能简单并基于Java进行配置

#### WebConfig.java

```java
package config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author ZhengTianle
 * Description:
 * 配置类，用于定义DispatcherServlet上下文的bean
 */
@Configuration
@EnableWebMvc                           //启用 Spring MVC
@ComponentScan("controller")            //扫描controller包
public class WebConfig extends WebMvcConfigurerAdapter {
    private static final Logger LOG = LoggerFactory.getLogger(WebConfig.class);

    /*配置jsp视图解析器*/
    @Bean
    public ViewResolver viewResolver(){
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("WEB-INF/view");
        resolver.setSuffix(".jsp");
        //可以在JSP页面中通过${}访问beans
        resolver.setExposeContextBeansAsAttributes(true);

        return resolver;
    }

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();//配置静态资源的处理
    }
}

```

#### RootConfig.java

```java
package config;

import com.alibaba.druid.pool.DruidDataSource;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.mybatis.spring.mapper.MapperScannerConfigurer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.jdbc.datasource.DriverManagerDataSource;

import javax.sql.DataSource;
import java.io.IOException;
import java.sql.SQLException;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author ZhengTianle
 * Description:
 * 主要配置持久层的一些东西，包括数据库、Mybatis框架，事务之类的东西
 */
@Configuration                                      //声明配置类
@ComponentScan({"dao", "service"})                  //扫描dao、service包
@PropertySource("classpath:db.properties")          //加载db.properties文件
//配置Mybatis扫包范围，从而为我们创建dao层的实现
@MapperScan(basePackages = "mapper", sqlSessionFactoryRef = "sqlSessionFactory")
public class RootConfig {

    private static final Logger LOG = LoggerFactory.getLogger(RootConfig.class);

    /**
     * 必须返回一个 PropertySourcesPlaceholderConfigurer 的bean,
     * 否则,会不能识别@Value("${userBean.name}") 注解中的 ${userBean.name}指向的value,
     * 而会注入${userBean.name}的字符串
     */
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer(){
        return new PropertySourcesPlaceholderConfigurer();
    }

    /*读取db.properities文件里面的值配置数据库*/
    @Bean
    public DataSource dataSource(@Value("${jdbc.driver}")String driver,
                                 @Value("${jdbc.url}")String url,
                                 @Value("${jdbc.username}")String username,
                                 @Value("${jdbc.password}")String password,
                                 @Value("${jdbc.initialSize}")int initialSize,
                                 @Value("${jdbc.minIdle}")int minIdle,
                                 @Value("${jdbc.maxActive}")int maxActive,
                                 @Value("${jdbc.maxWait}")int maxWait,
                                 @Value("${jdbc.timeBetweenEvictionRunsMills}")long timeBetweenEvictionRunsMillis,
                                 @Value("${jdbc.minEvictableIdleTimeMillis}")long minEvictableIdleTimeMillis,
                                 @Value("${jdbc.validationQuery}")String validationQuery,
                                 @Value("${jdbc.testWhileIdle}")boolean testWhileIdle,
                                 @Value("${jdbc.testOnBorrow}")boolean testOnBorrow,
                                 @Value("${jdbc.testOnReturn}")boolean testOnReturn,
                                 @Value("${jdbc.poolPreparedStatements}")boolean poolPreparedStatements,
                                 @Value("${jdbc.maxPoolPreparedStatementPerConnectionSize}")int maxPoolPreparedStatementPerConnectionSize,
                                 @Value("${jdbc.filters}")String filters,
                                 @Value("${jdbc.connectionProperties}")String connectionProperties){
        LOG.info("初始化Druid可监控式数据库连接池...");
        DruidDataSource dataSource = new DruidDataSource();

        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setDriverClassName(driver);

        //configuration
        dataSource.setInitialSize(initialSize);
        dataSource.setMinIdle(minIdle);
        dataSource.setMaxActive(maxActive);
        dataSource.setMaxWait(maxWait);
        dataSource.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
        dataSource.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
        dataSource.setValidationQuery(validationQuery);
        dataSource.setTestWhileIdle(testWhileIdle);
        dataSource.setTestOnBorrow(testOnBorrow);
        dataSource.setTestOnReturn(testOnReturn);
        dataSource.setPoolPreparedStatements(poolPreparedStatements);
        dataSource.setMaxPoolPreparedStatementPerConnectionSize(maxPoolPreparedStatementPerConnectionSize);
        try {
            dataSource.setFilters(filters);
        } catch (SQLException e) {
            LOG.error("druid配置初始化过滤器", e);
        }
        dataSource.setConnectionProperties(connectionProperties);
        return dataSource;
    }

    /*定义Mybatis的SessionFactory，配置mybatis-config.xml配置文件的位置信息*/
    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource) {
        //加载工程配置文件
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource);
        //如果不舍得xml配置的mybatis文件，额外的mybatis xml配置文件位置在这里配置
        factory.setConfigLocation(new ClassPathResource("mybatis-config.xml"));

        /**
         * 不舍得放弃mybatis xml mapper路径就在这里配置
         * 但是要在mapperXML文件夹里创建一个.xml
         * 否则如此配置会报FileNotFoundExceptiony异常
         */
        try {
            factory.setMapperLocations(resolver.getResources("classpath:mapperXml/*.xml"));
        } catch (IOException e) {
            LOG.error("映射mapper.xml", e);
        }
        //别名，让*Mpper.xml实体类映射可以不加上具体包名
        factory.setTypeAliasesPackage("mapper");

        return factory;
    }

    /*用来在Mybatis环境中控制数据库事务，使用方法：在Service方法上加上@Transactional即可*/
    @Bean
    public DataSourceTransactionManager dataSourceTransactionManager(DataSource dataSource){
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
        dataSourceTransactionManager.setDataSource(dataSource);

        return dataSourceTransactionManager;
    }

}

```
**注意**：单独的启用Spring MVC几乎不需要对`RootConfig.java`进行配置，仅仅标注一个`@Configuration`注解即可，但是这里为了尽量减少篇幅，直接把mybatis的配置放进去了

### 测试 Spring MVC

#### HomeController.java
```java
package controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.ModelAndView;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author ZhengTianle
 * Description:
 */
@Controller
public class HomeController {

    @RequestMapping(value = "/", method = RequestMethod.GET)    //处理对“/”的GET请求
    public String test(){
        return "home";          //视图名为home
    }
}

```

#### home.jsp

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>MVC</title>
</head>
<body>
    <h1>Hello Spring MVC</h1>

</body>
</html>

```

#### HomeControllerTest.java

> 从Spring3.2开始，我们可以按照控制器的方式来测试Spring MVC中的控制器了，而不仅仅是作为POJO进行测试。Spring现在包含了一种`mock Spring MVC`并针对控制器执行Http请求的机制。这样的话，在测试控制器的时候，就没有必要再启动Web服务器和Web浏览器了。

```java
package controller;

import config.WebConfig;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author ZhengTianle
 * Description:
 * 对HomeController.java的测试，验证Spring MVC启用成功与否
 */
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration//Web项目必须加上
@ContextConfiguration(classes = {WebConfig.class})
public class HomeControllerTest {

    @Autowired
    private HomeController homeController;

    @Test
    public void testHomePage() throws Exception {
        MockMvc mockMvc = MockMvcBuilders.standaloneSetup(homeController).build();//搭建MockMvc
        /**
         * 对“/”执行GET请求
         * 预期得到home视图
         */
        mockMvc.perform(get("/"))
                .andExpect(MockMvcResultMatchers.view().name("home"));

        System.out.println("预期正确！springmvc测试成功！");
    }
}
```
上述代码中发起了对"/"的 Get 请求，并断言结果视图的名称为home。它首先传递一个`HomeController`实例到`MockMvcBuilders.standaloneSetup()`并调用`build()`来构建`MockMvc`实例。然后它使用`MocMvc`实例来执行针对"/"的 Get 请求并设置期望得到`home`的视图名称。

**执行`testHomePage()`得到如下输出**
```shell
2018-11-29 12:23:54,703:INFO main (AbstractTestContextBootstrapper.java:248) - Loaded default TestExecutionListener class names from location [META-INF/spring.factories]: [org.springframework.test.context.web.ServletTestExecutionListener, org.springframework.test.context.support.DirtiesContextBeforeModesTestExecutionListener, org.springframework.test.context.support.DependencyInjectionTestExecutionListener, org.springframework.test.context.support.DirtiesContextTestExecutionListener, org.springframework.test.context.transaction.TransactionalTestExecutionListener, org.springframework.test.context.jdbc.SqlScriptsTestExecutionListener]
2018-11-29 12:23:54,750:INFO main (AbstractTestContextBootstrapper.java:177) - Using TestExecutionListeners: [org.springframework.test.context.web.ServletTestExecutionListener@7c729a55, org.springframework.test.context.support.DirtiesContextBeforeModesTestExecutionListener@3bb9a3ff, org.springframework.test.context.support.DependencyInjectionTestExecutionListener@661972b0, org.springframework.test.context.support.DirtiesContextTestExecutionListener@5af3afd9, org.springframework.test.context.transaction.TransactionalTestExecutionListener@323b36e0, org.springframework.test.context.jdbc.SqlScriptsTestExecutionListener@44ebcd03]2018-11-29 12:23:56,089:INFO main (MockServletContext.java:444) - Initializing Spring TestDispatcherServlet ''
2018-11-29 12:23:56,090:INFO main (FrameworkServlet.java:524) - Initializing Servlet ''
2018-11-29 12:23:56,091:INFO main (FrameworkServlet.java:546) - Completed initialization in 1 ms
预期正确！springmvc测试成功！

Process finished with exit code 0
```
**没有抛出任何异常，说明断言正确，Spring MVC测试成功**

## 整合 Mybatis

首先给出我导出的`test`数据库表sql语句：
```sql

CREATE TABLE `person` (
  `id` int(11) NOT NULL,
  `name` varchar(10) NOT NULL,
  `sex` tinyint(1) NOT NULL,
  `age` int(11) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- 转存表中的数据 `person`
INSERT INTO `person` (`id`, `name`, `sex`, `age`) VALUES
(1, 'fengbao', 1, 19),
(2, 'manjaro', 0, 20);

-- 表的索引 `person`
ALTER TABLE `person`
  ADD PRIMARY KEY (`id`);

-- 使用表AUTO_INCREMENT `person`
ALTER TABLE `person`
  MODIFY `id` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=3;
COMMIT;

```

### db.properties
```properties
jdbc.type=com.alibaba.druid.pool.DruidDataSource
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test?characterEncoding=utf-8
jdbc.username=username
jdbc.password=password

# 连接池设置
jdbc.initialSize=1
jdbc.minIdle=1
jdbc.maxActive=20
#连接等待超时时间
jdbc.maxWait=60000
# 配置间隔多久进行一次检测（检测可以关闭的空闲连接）
jdbc.timeBetweenEvictionRunsMills=60000
# 配置连接在池中最小生存时间
jdbc.minEvictableIdleTimeMillis=300000
jdbc.validationQuery=SELECT 1 FROM DUAL
jdbc.testWhileIdle=true
jdbc.testOnBorrow=false
jdbc.testOnReturn=false
# 打开PSCache，并指定每个连接上PSCache大小
jdbc.poolPreparedStatements=true
jdbc.maxPoolPreparedStatementPerConnectionSize=20
# 配置监控统计拦截的filters，去掉后监控界面sql无法统计
jdbc.filters=stat,wall,log4j2
# 打开mergeSql功能；慢SQL记录
jdbc.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
```

### generatorConfig.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--导入数据库配置信息-->
    <properties resource="db.properties" />

    <!--指定特定数据库的jdbc驱动jar包的位置-->
    <classPathEntry location="/home/zhengtianle/workspace/maven-repository/mysql/mysql-connector-java/8.0.13/mysql-connector-java-8.0.13.jar"/>

    <!--选择运行的mybatis版本-->
    <context id="default" targetRuntime="MyBatis3">
        <!--optional 设置没有默认注释，如果需要自定义注释，可以百度相关资料-->
        <commentGenerator>
            <property name="suppressDate" value="true" />
            <property name="suppressAllComments" value="true" />
        </commentGenerator>

        <!--jdbc数据库连接-->
        <jdbcConnection
                driverClass="${jdbc.driver}"
                connectionURL="${jdbc.url}"
                userId="${jdbc.username}"
                password="${jdbc.password}">
        </jdbcConnection>

        <!-- 配置数据库字段类型为DECIMAL 和 NUMERIC 是否强制解析成BigDecimal,可以不配置默认为false-->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="true"/>
        </javaTypeResolver>

        <!--Model模型生成器，用来生成含有主键key的类
        记录类以及查询Example类
        targetPackage：指定生成的model生成所在的包名
        targetProject：指定在该项目下所在的路径-->
        <javaModelGenerator targetPackage="pojo" targetProject="./src/main/java">
            <!--是否允许子包，即targetPackage.schemaName.tableName-->
            <property name="enableSubPackages" value="false" />
            <!-- 是否对model添加 构造函数 -->
            <property name="constructorBased" value="true"/>
            <!-- 是否对类CHAR类型的列的数据进行trim操作 -->
            <property name="trimStrings" value="true"/>
            <!-- 建立的Model对象是否不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
            <property name="immutable" value="false"/>
        </javaModelGenerator>

        <!--设置生成mapper.xml映射文件所在的目录&为每一个数据库的表生成对应的sqlMap文件-->
        <sqlMapGenerator targetPackage="mapperXml" targetProject="./src/main/resources">
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>

        <!--客户端代码，生成易于使用的针对Model对象和XML配置文件的代码
        type="ANNOTATEDMAPPER"：生成Java Model 和基于注解的Mapper对象
        type="MIXEDMAPPER"：生成基于注解的Java Model 和相应的Mapper对象
        type="XMLMAPPER"：生成SQLMap XML文件和独立的Mapper接口-->
        <!--生成mapper的包名和位置-->
        <javaClientGenerator type="ANNOTATEDMAPPER" targetPackage="mapper" targetProject="./src/main/java">
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>

        <!--mybatis generator代码生成器在默认的情况下会生成对于表实体类的一个Examle类,这里设置成false不生成Example-->
        <table tableName="person"
               domainObjectName="Person"
               enableCountByExample="false"
               enableUpdateByExample="false"
               enableDeleteByExample="false"
               enableSelectByExample="false"
               selectByExampleQueryId="false">

            <!--忽略列，不生成bean 字段
            <ignoreColumn column="FRED" />-->
            <!-- 指定列的java数据类型
            <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />-->
        </table>

    </context>
</generatorConfiguration>
```

### 生成pojo和Mapper文件

点击`idea`右边的`Maven Projects` -> `Maven Webapp` -> `Plugins` -> `mybatis-generator` -> `mybatis-generator:generate`,结果如下：
```shell
[INFO] 
[INFO] --- mybatis-generator-maven-plugin:1.3.7:generate (default-cli) @ ssm ---
[INFO] Connecting to the Database
[INFO] Introspecting table person
[INFO] Generating Record class for table person
[INFO] Generating Mapper Interface for table person
[INFO] Generating SQL Provider for table person
[INFO] Saving file Person.java
[INFO] Saving file PersonMapper.java
[INFO] Saving file PersonSqlProvider.java
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.913 s
[INFO] Finished at: 2018-11-29T12:47:14+08:00
[INFO] Final Memory: 15M/161M
[INFO] ------------------------------------------------------------------------

Process finished with exit code 0
```
- **在`PersonMapper.java`里面添加一个`list()`方法用以测试：**
```java
@Select("select * from person")
    List<Person> list();
```

- **在`Person.java`类上加一个`@ToString`注解以供测试**

### PersonMapperTest.java
```java
package mapper;

import config.Config;
import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import pojo.Person;

import java.util.List;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author ZhengTianle
 * Description:
 * mybatis简单的测试
 */
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration//Web项目必须加上
@ContextConfiguration(classes = {Config.class})
public class PersonMapperTest {

    @Autowired
    private PersonMapper personMapper;

    @Test
    public void test(){
        List<Person> personList = personMapper.list();
        personList.forEach(System.out::println);
    }

}

```
**运行`test()`得到如下结果**

```shell
2018-11-29 12:59:13,631:INFO main (AbstractTestContextBootstrapper.java:248) - Loaded default TestExecutionListener class names from location [META-INF/spring.factories]: [org.springframework.test.context.web.ServletTestExecutionListener, org.springframework.test.context.support.DirtiesContextBeforeModesTestExecutionListener, org.springframework.test.context.support.DependencyInjectionTestExecutionListener, org.springframework.test.context.support.DirtiesContextTestExecutionListener, org.springframework.test.context.transaction.TransactionalTestExecutionListener, org.springframework.test.context.jdbc.SqlScriptsTestExecutionListener]
2018-11-29 12:59:13,679:INFO main (AbstractTestContextBootstrapper.java:177) - Using TestExecutionListeners: [org.springframework.test.context.web.ServletTestExecutionListener@32eff876, org.springframework.test.context.support.DirtiesContextBeforeModesTestExecutionListener@8dbdac1, org.springframework.test.context.support.DependencyInjectionTestExecutionListener@6e20b53a, org.springframework.test.context.support.DirtiesContextTestExecutionListener@71809907, org.springframework.test.context.transaction.TransactionalTestExecutionListener@3ce1e309, org.springframework.test.context.jdbc.SqlScriptsTestExecutionListener@6aba2b86]2018-11-29 12:59:14,694:INFO main (RootConfig.java:71) - 初始化Druid可监控式数据库连接池...
2018-11-29 12:59:16,330:INFO main (DruidDataSource.java:947) - {dataSource-1} inited
Person(id=1, name=fengbao, sex=true, age=19)
Person(id=2, name=manjaro, sex=false, age=20)
2018-11-29 12:59:16,500:INFO Thread-1 (DruidDataSource.java:1863) - {dataSource-1} closed

Process finished with exit code 0
```
**拿到了我们数据库中的两行数据并放到了`personList`中，这说明我们成功利用mybatis和数据库交互了！**

## 附录

### log4j2.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" monitorInterval="30" strict="true" schema="Log4J-V2.2.xsd">
    <Appenders>
        <!-- 输出到控制台 -->
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss,SSS}:%4p %t (%F:%L) - %m%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info"> <!-- 全局配置 -->
            <AppenderRef ref="Console" />
        </Root>
    </Loggers>
</Configuration>
```

{% note danger no-icon %}

- 本文搭建的是一个极简版SSM框架，很多功能没有配置
- 本人第一次搭建SSM框架，在这个领域也是属于超级萌新，如果您看到文中哪一处不正确或者有更好的建议请联系本人邮箱或者留言，非常感谢！
- 最后感谢网友[四川-java后端-大神](http://118.24.158.160/)对本人的细心指导

{% endnote %}

**参考**：
Sping 实战(第4版)
[基于Spring4.x 搭建 Spring MVC + MyBatis](https://blog.csdn.net/captain_j/article/details/51407216)
[Spring下的Mybatis配置](https://www.jianshu.com/p/94951e80677a)
[SpringMVC Junit4测试 "No ServletContext set"](https://ask.csdn.net/questions/698500)
[SSM框架，基于JavaConfig配置方式，不用xml配置文件](https://blog.csdn.net/u012809062/article/details/73251036)
[Mybatis-Generator插件的使用与Spring集成Mybatis的配置](http://blog.51cto.com/zero01/2103687)



