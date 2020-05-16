---
title: SpringMVC02
toc: true
categories:
  - 后端
tags:
  - Spring
description: 分析Spring webMVC的启动流程
abbrlink: 88534fa4
date: 2020-05-12 22:36:40
---

# 分析 Spring webmvc 的启动流程

> Spring MVC 是 Spring 家族中比较有特点的一员，它分离了控制器、模型对象和视图层。Spring 的 MVC 框架是围绕 DispatcherServlet 设计的，主要由 DispatcherServlet、处理器映射、控制器、视图解析器、视图组成，Spring webmvc 启动以 web.xml 为入口，进行 web 初始化，再进行 Spring MVC 初始化。

<!--more-->

## 一、 web 初始化

### 1.1 从 xml 文件分析

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--1.注册监听器-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!--第一次初始化的内容-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>
    <!--2.过滤器，非必须，但过滤器监听器正常初始化-->
  <filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>
      org.springframework.web.filter.CharacterEncodingFilter
    </filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>utf-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <!--3. 初始化（注册）Servlet-->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>*.form</url-pattern>
    </servlet-mapping>
</web-app>
```

1. Instantiate an instance of each event listener identified by a <listener> element in the deployment descriptor.
2. For instantiated listener instances that _implement_ ServletContextListener, call the contextInitialized() method.
3. Instantiate an instance of each filter identified by a <filter> element in the deployment descriptor and call each filter instance's init() method.
4. Instantiate an instance of each servlet identified by a <servlet> element that includes a <load-on-startup> element in the order defined by the load-on-startup element values, and call each servlet instance's init() method.

- 上面是官网给出的 web 的**初始化**过程，首先注册**根监听器**，接着完成第一次初始化,即注册 applicationContext，并启动 Servlet**监听器**；然后初始化（注册）**过滤器**；再初始化（注册）**Servlet**。其中比较重要的是 ApplicationContext 注册和 Servlet 注册。
- ContextLoaderListener 类起着至关重要的作用。它读取 web.xml 中配置的 context-param 中的配置文件，提前在 web 容器初始化前准备业务对应的 Application context;**将创建好的 Application context 放置于 ServletContext 中**，为**springMVC**部分的初始化做好准备。
- 所以三个 xml 的加载顺序是 web.xml 创建 WebApplicationContext；
  ApplicationContext.xml 加载 bean；至此 web 就已经加载完了，然后是 DispatchServlet.xml，它会把前两个加载的东西拿来做参数，来初始化 DispatchServlet 管控 Spring MVC 部分。

### 1.2 从类的关系理解

- 最初始的类是 ContextLoader，实现了下面两个个方法：

```java
////创建父容器context，默认是XmlWebApplicationContext
public WebApplicationContext initWebApplicationContext(ServletContext servletContext)
......
protected WebApplicationContext createWebApplicationContext(ServletContext sc)
```

- ContextLoaderListener 类,继承了 ContextLoader。实现了 ServletContextListener。

### 2. IoC 容器创建

ApplicationContext 注册就是 IOC 容器的创建，实例化整个上下文中的 bean，所以 ApplicationContext.xml 是这样的：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

关于控制反转的概念，已经在上一篇博文中详细讲述，这里不再赘述，介绍几种创建 IoC 容器的方法：

```java
/**通过xml文件注册的bean的IoC创建*/
//classpath
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
//文件路径
ApplicationContext getApplicationContext = new FileSystemXmlApplicationContext("配置文件的绝对路径");
/**通过注解注册的bean的IoC容器创建*/
```

### 3. Bean 初始化

#### 3.1 bean 的概念

> 在 Spring 中，由 Spring IoC 容器管理的对象叫做 beans。 bean 就是由 Spring IoC 容器实例化、组装和以其他方式管理的对象。此外 bean 只是你应用中许多对象中的一个。Spring 框架中，一旦把一个 Bean 纳入到 Spring IoC 容器之中，这个 Bean 的生命周期就会交由容器进行管理。

#### 3.2 bean 的属性

| 属性                     | 描述                                                                                                        |
| :----------------------- | :---------------------------------------------------------------------------------------------------------- |
| class                    | 这个属性是强制性的，并且指定用来创建 bean 的 bean 类。                                                      |
| name                     | 这个属性指定唯一的 bean 标识符。在基于 XML 的配置元数据中，至少定义一个 ID 或 name 属性来指定 bean 标识符。 |
| scope                    | 这个属性指定由特定的 bean 定义创建的对象的作用域                                                            |
| constructor-arg          | 它是用来注入依赖关系的                                                                                      |
| properties               | 它是用来注入依赖关系的                                                                                      |
| autowiring mode          | 它是用来注入依赖关系的                                                                                      |
| lazy-initialization mode | 延迟初始化的 bean 告诉 IoC 容器在它第一次被请求时，而不是在启动时去创建一个 bean 实例。                     |
| initialization 方法      | 在 bean 的所有必需的属性被容器设置之后，调用回调方法。                                                      |
| destruction 方法         | 当包含该 bean 的容器被销毁时，使用回调方法。                                                                |

#### 3.3 bean 作用域

- singleton 作用域：默认的作用域。如果你想让 Spring 在每次需要时都返回同一个 bean 实例，你应该声明 bean 的作用域的属性为 singleton。
- prototype 作用域：为了强制 Spring 在每次需要时都产生一个新的 bean 实例，你应该声明 bean 的作用域的属性为 prototype。

#### 3.4 生命周期

- 实例化：实例化一个 Bean（由 BeanFactory 读取 Bean 定义文件，并生成各个实例）－－也就是我们常说的 new（@Bean）；
- 设置属性：按照 Spring 上下文对实例化的 Bean 的属性值进行设置。
- 使用初始化：调用 bean 的初始化方法。(将 bean 实例传递给 bean 后置处理器的 postProcessBeforeInitialization()方法)
- bean 可以使用了......
- 当容器关闭时，调用 bean 的销毁方法
  > 在配置 bean 时，通过 init-method 和 destroy-method 属性为 bean 指定初始化和销毁方法.

#### bean 注入的方式

1.  通过 xml 注入：

- 定义 bean

```java
@Data
public class Student {
    private String sno;
    private String sname;
    private String password;
}
```

- 在 ApplicationContext.xml 中注册

```xml
<bean id="student" class="Bean.Student" />
```

2. 通过注解注入：

- 首先在 spring 配置文件中开启注解扫描：

```xml
<context:component-scan base-package="org.example.spring.mvc"/>
```

- 使用注解声明 bean:

```java
@Component("jdbc")
public class StudentHomeWorkJdbc {
    //
}
```

3. 基于 java 类的配置

- 使用@Configuration 注解需要作为配置的类，表示该类将定义 Bean 的元数据
- 使用@Bean 注解相应的方法，该方法名默认就是 Bean 的名称，该方法返回值就是 Bean 的对象。
- AnnotationConfigApplicationContext 或子类进行加载基于 java 类的配置

```java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

## 二、 Spring MVC 启动流程

### 2.1 分析 DispatcherServlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!--此文件负责整个mvc中的配置-->

    <!--启用spring的一些annotation -->
    <context:annotation-config/>

    <!-- 配置注解驱动 可以将request参数与绑定到controller参数上 -->
    <mvc:annotation-driven/>

    <!--静态资源映射-->
    <!--本项目把静态资源放在了webapp目录下，资源映射如下-->
    <mvc:resources mapping="/assets/css/**" location="assets/css/"/>
    <mvc:resources mapping="/assets/fonts/**" location="assets/fonts/"/>
    <mvc:resources mapping="/assets/img/**" location="assets/img/"/>
    <mvc:resources mapping="/assets/js/**" location="assets/js/"/>

    <!-- 对模型视图名称的解析，即在模型视图名称添加前后缀(如果最后一个还是表示文件夹,则最后的斜杠不要漏了) 使用JSP-->
    <!-- 默认的视图解析器 在上边的解析错误时使用 (默认使用html)- -->
    <bean id="defaultViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/"/><!--设置JSP文件的目录位置-->
        <property name="suffix" value=".jsp"/>
        <property name="exposeContextBeansAsAttributes" value="true"/>
    </bean>

    <!--自动扫描装配 （base-package由包名决定-->
    <context:component-scan base-package="Controller"/>
</beans>
```

### 2.2 图解

{% asset_img SpringMVC.png  %}

1.  客户端请求提交到 DispatcherServlet。
2.  由 DispatcherServlet 控制器寻找一个或多个 HandlerMapping，找到处理请求的 Controller。
3.  DispatcherServlet 将请求提交到 Controller。
4.  Controller 调用业务逻辑处理后返回 ModelAndView。
5.  DispatcherServlet 寻找一个或多个 ViewResolver 视图解析器，找到 ModelAndView 指定的视图。
6.  视图负责将结果显示到客户端。
    > 共包含 4 个 Spring MVC 接口，即 DispatcherServlet、HandlerMapping、Controller 和 ViewResolver。

- Spring MVC 所有的请求都经过 DispatcherServlet 来统一分发，在 DispatcherServlet 将请求分发给 Controller 之前需要借助 Spring MVC 提供的 HandlerMapping 定位到具体的 Controller。

- HandlerMapping 接口负责完成客户请求到 Controller 映射。

- Controller 接口将处理用户请求，这和 Java Servlet 扮演的角色是一致的。一旦 Controller 处理完用户请求，将返回 ModelAndView 对象给 DispatcherServlet 前端控制器，ModelAndView 中包含了模型（Model）和视图（View）。

- 从宏观角度考虑，DispatcherServlet 是整个 Web 应用的控制器；从微观考虑，Controller 是单个 Http 请求处理过程中的控制器，而 ModelAndView 是 Http 请求过程中返回的模型（Model）和视图（View）。

- ViewResolver 接口（视图解析器）在 Web 应用中负责查找 View 对象，从而将相应结果渲染给客户。
