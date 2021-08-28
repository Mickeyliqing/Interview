# Spring

## Spring Bean 的生命周期

1. Bean 的定义。
2. Bean 的初始化。
3. Bean 的调用。
4. Bean 的销毁。



## Spring 属性注入的方式

1. 值类型注入。@Value() 注解来完成。可以声明在对应的属性上或者方法上。
2. 引用类型的注入。@Autowired。@Autowired 和 @Qualifier() 二者结合。@Resource()。
3. @Autowired 注解是 JDK 的。它按照类型装配依赖对象，默认情况下依赖对象必须存在。如果为 NULL ，则 request 的属性为 false。
4. @Resource() 是 Spring 的。它默认按照名称装配对象，名称通过 name 属性指定。



## Spring 中的事务

**定义：**

事务是对一系列数据的操作进行统一的提交和回滚操作。如果成功则一起成功，如果中间有一条失败，那么回滚之前所有的操作。



**操作方法：**

开启事务，提交事务，回滚事务。



## Spring 将对象注册到容器中的方法

1. @Component()。
2. @Service()。
3. @Controller()。
4. @Respository()。



## Spring AOP

aop : 面向切面编程是一种编程思想，也就是说要实现将功能代码从业务代码里面分离。其中拦截器就可以看作是面向切面编程思想的一种实现。

底层是动态代理机制，其中分为：JDK 动态代理和 CGlib 动态代理。

其中 JDK 动态代理主要是对实现了接口的类进行动态代理。

CGlib 主要是类实现动态代理。



## Spring IOC 

ioc : 是一种设计思想，将原本在程序中手动创建对象的控制权交给了 Spring 框架来管理。

# SpringBoot

## SpringBoot 怎么配多数据源

1. 在配置文件中配置多个数据源，然后通过配置类（获取不同数据源的不同的类）来获取数据源以及 Mapper 相关配置。
2. 配置一个默认使用的数据源，然后定义多个其他的数据源，使用 Aop 形式注解选择数据源。



## SpringBoot 的核心注解

1. @SpringBootApplication ：用在 SpringBoot 主类上。其实是  @SpringBootConfiguration，@EnableAutoConfiguration ，@ComponentScan 这三个注解的组合。
2. @SpringBootConfiguration ：@Configuration 注解的变体。
3. @EnableAutoConfiguration ：SpringBoot 的自动配置注解。
4. @Configuration ：SpringBoot 用来代替 applicationContext.xml 配置文件的一个注解。
5. @ComponentScan ：开启组件扫描。
6. @Conditional ：SpringBoot 用来标识一个 Bean 或者 Configuration 配置文件 。
7. @Import ：SpringBoot 表示一个或者多个 @Configuration 注解修饰的类 。



## SpringBoot 核心配置文件

1. application ：SpringBoot 项目的自动化配置。
2. bootstrap  ：使用 SpringCloud Config 配置中心时，这时需要在 bootstrap 配置文件中添加连接到配置中心的配置属性来加载外部配置中心的配置信息； 一些固定的不能被覆盖的属性；一些加密/解密的场景 。



## SpringBoot 自动配置原理

注解 @EnableAutoConfiguration, @Configuration, @ConditionalOnClass 就是自动配置的核心，首先它得是一个配置文件，其次根据类路径下是否有这个类去自动配置。 



## SpringBoot 如何实现分页和排序的

使用的是 Spring Data 提供的 Pageable、Sort、Page 类 。



## SpringBoot 是如何处理异常的

1. 自定义异常处理。
2. 使用 @ExceptionHandler 注解处理局部异常。
3. 使用 @ControllerAdvice + @ExceptionHandler 来处理全局异常。
4. 通过实现 HandlerExceptionResolver 接口处理异常 。



## RequestMapping 和 GetMapping 的不同之处

1. RequestMapping 具有类属性的，可以进行 GET,POST,PUT 或者其它的注释中具有的请求方法。 
2. GetMapping 是 GET 请求方法中的一个特例。



## SpringBoot 的加载过程

SpringBoot 在启动的时候，按照约定会去读取 SpringBoot Starter 的配置信息，会根据配置信息对资源进行初始化，并注入 Spring 容器中，这样当 Spring 启动好之后，就已经准备好一切资源，使用过程直接注入对应的 Bean 即可。



## Spring 和 SpringMVC 和 SpringBoot

1. Spring 是一个轻量级的 JAVA 框架，核心是控制反转和面向切面编程。
2. SpringMVC 是Spring 基础上的一个MVC 框架，主要使用于 WEB 开发。
3. SpringBoot 是在 SpringMVC 基础上，根据默认大于配置的思想，简化配置，集成快速开实现的一个框架，常用于微服务系统开发。

# MyBatis

## MyBatis 是如何进行分页的

MyBatis 使用 RowBounds 对象进行分页。针对结果集执行内存分页，可以在 SQL 中直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成。



## MyBatis 中 #{} 和 ${} 的区别

1. #{} 是预编译处理，传进来的数据会加上 “ ”。
2. ${} 是字符串替换，直接替换到占位符，使用的话会导致 SQL 注入的风险。

