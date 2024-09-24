---
title: SpringMVC请求流程设计
tags:
  - JavaWeb
categories:
  - Java
date: 2024-05-15 17:48:00
---

# Spring基础 - SpringMVC请求流程和案例 

# [#](#spring%E5%9F%BA%E7%A1%80-springmvc%E8%AF%B7%E6%B1%82%E6%B5%81%E7%A8%8B%E5%92%8C%E6%A1%88%E4%BE%8B) Spring基础 - SpringMVC请求流程和案例

> 前文我们介绍了Spring框架和Spring框架中最为重要的两个技术点（IOC和AOP），那我们如何更好的构建上层的应用呢（比如web 应用），这便是SpringMVC；Spring MVC是Spring在Spring Container Core和AOP等技术基础上，遵循上述Web MVC的规范推出的web开发框架，目的是为了简化Java栈的web开发。 本文主要介绍SpringMVC主要的流程和基础案例的编写和运行。

## [#](#%E5%BC%95%E5%85%A5) 引入

> 前文我们介绍了Spring框架和Spring框架中最为重要的两个技术点（IOC和AOP），同时我们也通过几个Demo应用了Core Container中包

那么问题是，我们如何在Core Container的基础上更好的构建上层的应用呢？比如web 应用。

针对上层的Web应用，SpringMVC诞生了，它也是Spring技术栈中最为重要的一个框架。

**所以为了更好的帮助你串联整个知识体系，我列出了几个问题，通过如下几个问题帮你深入浅出的构建对SpringMVC的认知**。

*   Java技术栈的Web应用是如何发展的？
*   什么是MVC，什么是SpringMVC？
*   SpringMVC主要的请求流程是什么样的？
*   SpringMVC中还有哪些组件？
*   如何编写一个简单的SpringMVC程序呢？

## [#](#%E4%BB%80%E4%B9%88%E6%98%AFmvc) 什么是MVC

> MVC英文是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计规范。本质上也是一种解耦。

用一种把**用户交互逻辑【Controller】、业务数据处理逻辑【Model】、界面显示【View】**分离的方法，根据职责解耦。将业务数据处理逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。MVC被独特的发展起来用于映射传统的输入、处理和输出功能在一个逻辑的图形化用户界面的结构中。

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-08d1106a7dc878b9956f6ca994c45154.png)

*   **Model**（模型）是应用程序中用于处理应用程序数据逻辑的部分。通常模型对象负责在数据库中存取数据。Service层、DAO层都可以看作Model的组成部分。
*   **View**（视图）是应用程序中处理数据显示的部分。通常视图是依据模型数据创建的。
*   **Controller**（控制器）是应用程序中处理用户交互的部分。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。

## [#](#%E4%BB%80%E4%B9%88%E6%98%AFspring-mvc) 什么是Spring MVC

> 简单而言，Spring MVC是Spring在Spring IOC和AOP等技术基础上，遵循上述Web MVC的规范推出的web开发框架，目的是为了简化Java栈的web开发。

Spring Web MVC 是一种基于Java 的实现了Web MVC 设计模式的请求驱动类型的轻量级Web 框架，即使用了MVC架构模式的思想，将 web 层进行职责解耦，基于请求驱动指的就是使用请求-响应模型，框架的目的就是帮助我们简化开发，Spring Web MVC 也是要简化我们日常Web开发的。

> 小贴士
>
> 使用原生的Servlet开发，所有代码糅合在一起，难以维护。MVC根据职责把代码分类，交给不同的组件负责。
>
> - 参数解析处理、内部转发、重定向等属于用户交互逻辑，交给Controller。
> - 业务数据处理逻辑，交给Model
> - 视图渲染，交给View

**相关特性如下**：

*   让我们能非常简单的设计出干净的Web 层和薄薄的Web 层；
*   进行更简洁的Web 层的开发；
*   天生与Spring 框架集成（如IoC 容器、AOP 等）；
*   提供强大的约定大于配置的契约式编程支持；
*   能简单的进行Web 层的单元测试；
*   支持灵活的URL 到页面控制器的映射；
*   非常容易与其他视图技术集成，如 Velocity、FreeMarker 等等，因为模型数据不放在特定的 API 里，而是放在一个 Model 里（Map 数据结构实现，因此很容易被其他框架使用）；
*   非常灵活的数据验证、格式化和数据绑定机制，能使用任何对象进行数据绑定，不必实现特定框架的API；
*   提供一套强大的JSP 标签库，简化JSP 开发；
*   支持灵活的本地化、主题等解析；
*   更加简单的异常处理；
*   对静态资源的支持；
*   支持Restful 风格。

## [#](#spring-mvc%E7%9A%84%E8%AF%B7%E6%B1%82%E6%B5%81%E7%A8%8B) Spring MVC的请求流程

> Spring Web MVC 框架也是一个基于请求驱动的Web 框架，并且也使用了前端控制器模式来进行设计实现，再根据请求映射规则分发给相应的页面控制器（动作/处理器）进行处理。

> MVC实现了按职责模块间的解耦，前端控制器模式是为了避免硬编码、复用代码、屏蔽底层方法调用【比如Servlet规范中的请求、响应对象】等目标。

### [#](#%E6%A0%B8%E5%BF%83%E6%9E%B6%E6%9E%84%E7%9A%84%E5%85%B7%E4%BD%93%E6%B5%81%E7%A8%8B%E6%AD%A5%E9%AA%A4) 核心架构的具体流程步骤

> 首先让我们整体看一下Spring Web MVC 处理请求的流程：

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-91907ed8ef56da64b7bc0ce99073a0e1.png)

**核心架构的具体流程步骤**如下：

1.  **首先用户发送请求——>DispatcherServlet**，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行 处理，作为统一访问点，进行全局的流程控制；
2.  **DispatcherServlet——>HandlerMapping**， HandlerMapping 将会把请求映射为 HandlerExecutionChain 对象（包含一 个Handler 处理器（页面控制器）对象、多个HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新 的映射策略；
3.  **DispatcherServlet——>HandlerAdapter**，HandlerAdapter 将会把处理器包装为适配器，从而支持多种类型的处理器， 即适配器设计模式的应用，从而很容易支持很多类型的处理器；
4.  **HandlerAdapter——>处理器功能处理方法的调用**，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处 理方法，完成功能处理；并返回一个ModelAndView 对象（包含模型数据、逻辑视图名）；
5.  **ModelAndView 的逻辑视图名——> ViewResolver**，ViewResolver 将把逻辑视图名解析为具体的View，通过策略模式，很容易更换其他视图技术；
6.  **View——>渲染**，View 会根据传进来的Model 模型数据进行渲染，此处的Model 实际是一个Map 数据结构，因此 很容易支持其他视图技术；
7.  **返回控制权给DispatcherServlet**，由DispatcherServlet 返回响应给用户，到此一个流程结束。

### [#](#%E5%AF%B9%E4%B8%8A%E8%BF%B0%E6%B5%81%E7%A8%8B%E7%9A%84%E8%A1%A5%E5%85%85) 对上述流程的补充

> 上述流程只是核心流程，这里我们再补充一些其它组件：

1.  **Filter(ServletFilter)**

进入Servlet前可以有preFilter, Servlet处理之后还可有postFilter

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-74c793c33383c0a31e7e6c0dc7caba1b.png)

2.  **LocaleResolver**

在视图解析/渲染时，还需要考虑国际化(Local)，显然这里需要有LocaleResolver.

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-575f898a8b784fef95b98d5b5c76d4a6.png)

3.  **ThemeResolver**

如何控制视图样式呢？SpringMVC中还设计了ThemeSource接口和ThemeResolver，包含一些静态资源的集合(样式及图片等），用来控制应用的视觉风格。

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-3777b04e27ac07ca7100a842aca8ff0a.png)

4.  **对于文件的上传请求**？

对于常规请求上述流程是合理的，但是如果是文件的上传请求，那么就不太一样了；所以这里便出现了MultipartResolver。

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-42ce331a2d8d6bbbb3eb4aae20dc12e9.png)

## [#](#spring-mvc%E6%A1%88%E4%BE%8B) Spring MVC案例

> 这里主要向你展示一个基本的SpringMVC例子，后文中将通过以Debug的方式分析源码。

本例子中主要文件和结构如下：

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-f2f8bf829a328343d4510ca006662fba.png)

### [#](#maven%E5%8C%85%E5%BC%95%E5%85%A5) Maven包引入

主要引入spring-webmvc包（spring-webmvc包中已经包含了Spring Core Container相关的包），以及servlet和jstl（JSP中使用jstl)的包。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tech-pdai-spring-demos</artifactId>
        <groupId>tech.pdai</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>011-spring-framework-demo-springmvc</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <spring.version>5.3.9</spring.version>
        <servlet.version>4.0.1</servlet.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>${servlet.version}</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
        </dependency>
    </dependencies>

</project>
```

### [#](#%E4%B8%9A%E5%8A%A1%E4%BB%A3%E7%A0%81%E7%9A%84%E7%BC%96%E5%86%99) 业务代码的编写

User实体

```java
package tech.pdai.springframework.springmvc.entity;

/**
 * @author pdai
 */
public class User {

    /**
     * user's name.
     */
    private String name;

    /**
     * user's age.
     */
    private int age;

    /**
     * init.
     *
     * @param name name
     * @param age  age
     */
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

Dao

```java
package tech.pdai.springframework.springmvc.dao;

import org.springframework.stereotype.Repository;
import tech.pdai.springframework.springmvc.entity.User;

import java.util.Collections;
import java.util.List;

/**
 * @author pdai
 */
@Repository
public class UserDaoImpl {

    /**
     * mocked to find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return Collections.singletonList(new User("pdai", 18));
    }
}
```

Service

```java
package tech.pdai.springframework.springmvc.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import tech.pdai.springframework.springmvc.dao.UserDaoImpl;
import tech.pdai.springframework.springmvc.entity.User;

import java.util.List;

/**
 * @author pdai
 */
@Service
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    @Autowired
    private UserDaoImpl userDao;

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return userDao.findUserList();
    }

}
```

Controller

```java
package tech.pdai.springframework.springmvc.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import tech.pdai.springframework.springmvc.service.UserServiceImpl;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Date;

/**
 * User Controller.
 *
 * @author pdai
 */
@Controller
public class UserController {

    @Autowired
    private UserServiceImpl userService;

    /**
     * find user list.
     *
     * @param request  request
     * @param response response
     * @return model and view
     */
    @RequestMapping("/user")
    public ModelAndView list(HttpServletRequest request, HttpServletResponse response) {
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("dateTime", new Date());
        modelAndView.addObject("userList", userService.findUserList());
        modelAndView.setViewName("userList"); // views目录下userList.jsp
        return modelAndView;
    }
}
```

### [#](#webapp%E4%B8%8B%E7%9A%84web-xml) webapp下的web.xml

（创建上图的文件结构）

webapp下的web.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <display-name>SpringFramework - SpringMVC Demo @pdai</display-name>

    <servlet>
        <servlet-name>springmvc-demo</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 通过初始化参数指定SpringMVC配置文件的位置和名称 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>springmvc-demo</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

### [#](#springmvc-xml) springmvc.xml

web.xml中我们配置初始化参数contextConfigLocation，路径是classpath:springmvc.xml

```xml
<init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc.xml</param-value>
</init-param>
```

在resources目录下创建

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 扫描注解 -->
    <context:component-scan base-package="tech.pdai.springframework.springmvc"/>

    <!-- 静态资源处理 -->
    <mvc:default-servlet-handler/>

    <!-- 开启注解 -->
    <mvc:annotation-driven/>

    <!-- 视图解析器 -->
    <bean id="jspViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

### [#](#jsp%E8%A7%86%E5%9B%BE) JSP视图

创建userList.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <title>User List</title>

    <!-- Bootstrap -->
    <link rel="stylesheet" href="//cdn.bootcss.com/bootstrap/3.3.5/css/bootstrap.min.css">

</head>
<body>
    <div class="container">
        <c:if test="${!empty userList}">
            <table class="table table-bordered table-striped">
                <tr>
                    <th>Name</th>
                    <th>Age</th>
                </tr>
                <c:forEach items="${userList}" var="user">
                    <tr>
                        <td>${user.name}</td>
                        <td>${user.age}</td>
                    </tr>
                </c:forEach>
            </table>
        </c:if>
    </div>
</body>
</html>
```

### [#](#%E9%83%A8%E7%BD%B2%E6%B5%8B%E8%AF%95) 部署测试

> 我们通过IDEA的tomcat插件来进行测试

下载Tomcat：[tomcat地址在新窗口打开](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/https://downloads.apache.org/tomcat_Img/)

下载后给tomcat/bin执行文件赋权

```bash
pdai@MacBook-Pro pdai % cd apache-tomcat-9.0.62 
pdai@MacBook-Pro apache-tomcat-9.0.62 % cd bin 
pdai@MacBook-Pro bin % ls
bootstrap.jar			makebase.sh
catalina-tasks.xml		setclasspath.bat
catalina.bat			setclasspath.sh
catalina.sh			shutdown.bat
ciphers.bat			shutdown.sh
ciphers.sh			startup.bat
commons-daemon-native.tar.gz	startup.sh
commons-daemon.jar		tomcat-juli.jar
configtest.bat			tomcat-native.tar.gz
configtest.sh			tool-wrapper.bat
daemon.sh			tool-wrapper.sh
digest.bat			version.bat
digest.sh			version.sh
makebase.bat
pdai@MacBook-Pro bin % chmod 777 *.sh
pdai@MacBook-Pro bin % 
```

配置Run Congfiuration

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-b841c83be4671839241bf10ccf144ffd.png)

添加Tomcat Server - Local

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-3e80e623ba76ce6dd74cd3b12e4cd4cc.png)

将我们下载的Tomcat和Tomcat Server - Local关联

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-07b962a61596710a96bbfc16294cec21.png)

在Deploy中添加我们的项目

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-c58da10c77504239ef18ab46d9e71e64.png)

运行和管理Tomcat Sever（注意context路径）

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-9a1e4d3cd836a3f8972f78eeebd7f3d4.png)

运行后访问我们的web程序页面（注意context路径）

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723125738-0c68cde34b6e9626314e402ff24a19c0.png)

PS：是不是so easy~ 

## [#](#%E7%A4%BA%E4%BE%8B%E6%BA%90%E7%A0%81) 示例源码

https://github.com/realpdai/tech-pdai-spring-demos





# DispatcherServlet的初始化过程 

# [#](#spring%E8%BF%9B%E9%98%B6-springmvc%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E4%B9%8Bdispatcherservlet%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96%E8%BF%87%E7%A8%8B) Spring进阶 - SpringMVC实现原理之DispatcherServlet的初始化过程

> 前文我们有了IOC的源码基础以及SpringMVC的基础，我们便可以进一步深入理解SpringMVC主要实现原理，包含DispatcherServlet的初始化过程和DispatcherServlet处理请求的过程的源码解析。本文是第一篇：DispatcherServlet的初始化过程的源码解析。

## [#](#dispatcherservlet%E5%92%8Capplicationcontext%E6%9C%89%E4%BD%95%E5%85%B3%E7%B3%BB) DispatcherServlet和ApplicationContext有何关系？

与许多其他web框架一样，Spring MVC是围绕前端控制器模式设计的，其中DispatcherServlet作为中心Servlet ，为请求处理提供共享算法，由可配置的委托组件执行实际工作。这个模型是灵活的，并且支持不同的工作流。

DispatcherServlet和任何Servlet一样，需要通过使用Java配置或在web.xml中根据Servlet规范声明和映射。相应的，DispatcherServlet使用Spring配置来发现请求映射、视图解析、异常处理等所需的委托组件。

> 那DispatcherServlet和ApplicationContext有和关系呢？如下内容可以参考[官网-SpringMVC文档在新窗口打开](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/https://docs.spring.io/spring-framework/docs/current/reference/html_Img/web.html#mvc-servlet)

DispatcherServlet 需要一个 WebApplicationContext（普通ApplicationContext的扩展） 来管理配置自己的组件。WebApplicationContext 可以链接到ServletContext和它自己所关联的Servlet。因为绑定到了ServletContext，这样应用程序就可以在需要的时候使用 RequestContextUtils 的静态方法访问 WebApplicationContext。

对大多数应用程序来说，配置一个WebApplicationContext既简单也满足需求。除此之外，也可以配置一个上下文树，一个Root WebApplicationContext 被多个 Servlet实例共享，每个Servlet又有自己的Servlet WebApplicationContext 配置。

Root WebApplicationContext 通常包含需要共享给多个 Servlet 实例的基础 Bean，比如数据源对象和业务服务对象。这些 Bean 实际上被继承了，而且可以在 Servlet专属的子WebApplicationContext中覆盖【重新声明】。

（PS：官网上的这张图可以可以帮助你构建DispatcherServlet和ApplicationContext在设计上的认知，这一点对于理解DispatcherServlet的设计和初始化过程非常重要）

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723260289-9708aa67f67bc662706974ba0f755453-1723260470321-1.png)

## [#](#dispatcherservlet%E6%98%AF%E5%A6%82%E4%BD%95%E5%88%9D%E5%A7%8B%E5%8C%96%E7%9A%84) DispatcherServlet是如何初始化的？

> DispatcherServlet首先是Sevlet，Servlet有自己的生命周期的方法（init,destory等），那么我们在看DispatcherServlet初始化时首先需要看源码中DispatcherServlet的类结构设计。

首先我们看DispatcherServlet的类结构关系，在这个类依赖结构中找到init的方法

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723260289-2b8657b9df0f1bfc9c7a8ce60aae4e8a-1723260470322-3.png)

很容易找到init()的方法位于HttpServletBean中，然后跑[Spring基础 - SpringMVC请求流程和案例](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/https://pdai.tech/md/spring_Img/spring-x-framework-springmvc.html)中的代码，在init方法中打断点。

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723260289-5469bc677d465a56b6f0cc03ee1b43ae-1723260470322-4.png)

### [#](#init) init

init()方法如下, 主要读取web.xml中servlet参数配置，并将交给子类方法initServletBean()继续初始化

```java
/**
  * Map config parameters onto bean properties of this servlet, and
  * invoke subclass initialization.
  * @throws ServletException if bean properties are invalid (or required
  * properties are missing), or if subclass initialization fails.
  */
@Override
public final void init() throws ServletException {

  // 读取web.xml中的servlet配置
  PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
  if (!pvs.isEmpty()) {
    try {
      // 转换成BeanWrapper，为了方便使用Spring的属性注入功能
      BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
      // 注入Resource类型需要依赖于ResourceEditor解析，所以注册Resource类关联到ResourceEditor解析器
      ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
      bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
      // 更多的初始化可以让子类去拓展
      initBeanWrapper(bw);
      // 让spring注入namespace,contextConfigLocation等属性
      bw.setPropertyValues(pvs, true);
    }
    catch (BeansException ex) {
      if (logger.isErrorEnabled()) {
        logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
      }
      throw ex;
    }
  }

  // 让子类去拓展
  initServletBean();
}
```

读取配置可以从下图看出，正是初始化了我们web.xml中配置

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723260289-adf1d1788cf1c508c5cfc9a316446527-1723260470322-2.png)

再看下initServletBean()方法，位于FrameworkServlet类中

```java
/**
  * Overridden method of {@link HttpServletBean}, invoked after any bean properties
  * have been set. Creates this servlet's WebApplicationContext.
  */
@Override
protected final void initServletBean() throws ServletException {
  getServletContext().log("Initializing Spring " + getClass().getSimpleName() + " '" + getServletName() + "'");
  if (logger.isInfoEnabled()) {
    logger.info("Initializing Servlet '" + getServletName() + "'");
  }
  long startTime = System.currentTimeMillis();

  try {
    // 最重要的是这个方法
    this.webApplicationContext = initWebApplicationContext();

    // 可以让子类进一步拓展
    initFrameworkServlet();
  }
  catch (ServletException | RuntimeException ex) {
    logger.error("Context initialization failed", ex);
    throw ex;
  }

  if (logger.isDebugEnabled()) {
    String value = this.enableLoggingRequestDetails ?
        "shown which may lead to unsafe logging of potentially sensitive data" :
        "masked to prevent unsafe logging of potentially sensitive data";
    logger.debug("enableLoggingRequestDetails='" + this.enableLoggingRequestDetails +
        "': request parameters and headers will be " + value);
  }

  if (logger.isInfoEnabled()) {
    logger.info("Completed initialization in " + (System.currentTimeMillis() - startTime) + " ms");
  }
}
```

### [#](#initwebapplicationcontext) initWebApplicationContext

initWebApplicationContext用来初始化和刷新WebApplicationContext。

initWebApplicationContext() 方法如下

```java
/**
  * Initialize and publish the WebApplicationContext for this servlet.
  * <p>Delegates to {@link #createWebApplicationContext} for actual creation
  * of the context. Can be overridden in subclasses.
  * @return the WebApplicationContext instance
  * @see #FrameworkServlet(WebApplicationContext)
  * @see #setContextClass
  * @see #setContextConfigLocation
  */
protected WebApplicationContext initWebApplicationContext() {
  WebApplicationContext rootContext =
      WebApplicationContextUtils.getWebApplicationContext(getServletContext());
  WebApplicationContext wac = null;

  // 如果在构造函数已经被初始化
  if (this.webApplicationContext != null) {
    // A context instance was injected at construction time -> use it
    wac = this.webApplicationContext;
    if (wac instanceof ConfigurableWebApplicationContext) {
      ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
      if (!cwac.isActive()) {
        // The context has not yet been refreshed -> provide services such as
        // setting the parent context, setting the application context id, etc
        if (cwac.getParent() == null) {
          // The context instance was injected without an explicit parent -> set
          // the root application context (if any; may be null) as the parent
          cwac.setParent(rootContext);
        }
        configureAndRefreshWebApplicationContext(cwac);
      }
    }
  }
  // 没有在构造函数中初始化，则尝试通过contextAttribute初始化
  if (wac == null) {
    // No context instance was injected at construction time -> see if one
    // has been registered in the servlet context. If one exists, it is assumed
    // that the parent context (if any) has already been set and that the
    // user has performed any initialization such as setting the context id
    wac = findWebApplicationContext();
  }

  // 还没有的话，只能重新创建了
  if (wac == null) {
    // No context instance is defined for this servlet -> create a local one
    wac = createWebApplicationContext(rootContext);
  }

  if (!this.refreshEventReceived) {
    // Either the context is not a ConfigurableApplicationContext with refresh
    // support or the context injected at construction time had already been
    // refreshed -> trigger initial onRefresh manually here.
    synchronized (this.onRefreshMonitor) {
      onRefresh(wac);
    }
  }

  if (this.publishContext) {
    // Publish the context as a servlet context attribute.
    String attrName = getServletContextAttributeName();
    getServletContext().setAttribute(attrName, wac);
  }

  return wac;
}
```

webApplicationContext只会初始化一次，依次尝试构造函数初始化，没有则通过contextAttribute初始化，仍没有则创建新的

创建的createWebApplicationContext方法如下

```java
/**
  * Instantiate the WebApplicationContext for this servlet, either a default
  * {@link org.springframework.web.context.support.XmlWebApplicationContext}
  * or a {@link #setContextClass custom context class}, if set.
  * <p>This implementation expects custom contexts to implement the
  * {@link org.springframework.web.context.ConfigurableWebApplicationContext}
  * interface. Can be overridden in subclasses.
  * <p>Do not forget to register this servlet instance as application listener on the
  * created context (for triggering its {@link #onRefresh callback}, and to call
  * {@link org.springframework.context.ConfigurableApplicationContext#refresh()}
  * before returning the context instance.
  * @param parent the parent ApplicationContext to use, or {@code null} if none
  * @return the WebApplicationContext for this servlet
  * @see org.springframework.web.context.support.XmlWebApplicationContext
  */
protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
  Class<?> contextClass = getContextClass();
  if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
    throw new ApplicationContextException(
        "Fatal initialization error in servlet with name '" + getServletName() +
        "': custom WebApplicationContext class [" + contextClass.getName() +
        "] is not of type ConfigurableWebApplicationContext");
  }

  // 通过反射方式初始化
  ConfigurableWebApplicationContext wac =
      (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);

  wac.setEnvironment(getEnvironment());
  wac.setParent(parent);
  String configLocation = getContextConfigLocation(); // 就是前面Demo中的springmvc.xml
  if (configLocation != null) {
    wac.setConfigLocation(configLocation);
  }

  // 初始化Spring环境
  configureAndRefreshWebApplicationContext(wac);

  return wac;
}
```

configureAndRefreshWebApplicationContext方法初始化设置Spring环境

```java
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
  // 设置context ID
  if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
    // The application context id is still set to its original default value
    // -> assign a more useful id based on available information
    if (this.contextId != null) {
      wac.setId(this.contextId);
    }
    else {
      // Generate default id...
      wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
          ObjectUtils.getDisplayString(getServletContext().getContextPath()) + '/' + getServletName());
    }
  }

  // 设置servletContext, servletConfig, namespace, listener...
  wac.setServletContext(getServletContext());
  wac.setServletConfig(getServletConfig());
  wac.setNamespace(getNamespace());
  wac.addApplicationListener(new SourceFilteringListener(wac, new ContextRefreshListener()));

  // The wac environment's #initPropertySources will be called in any case when the context
  // is refreshed; do it eagerly here to ensure servlet property sources are in place for
  // use in any post-processing or initialization that occurs below prior to #refresh
  ConfigurableEnvironment env = wac.getEnvironment();
  if (env instanceof ConfigurableWebEnvironment) {
    ((ConfigurableWebEnvironment) env).initPropertySources(getServletContext(), getServletConfig());
  }

  // 让子类去拓展
  postProcessWebApplicationContext(wac);
  applyInitializers(wac);

  // Spring环境初始化完了，就可以初始化DispatcherServlet处理流程中需要的组件了。
  wac.refresh();
}
```

### [#](#refresh) refresh

有了webApplicationContext后，就开始刷新了（onRefresh()方法），这个方法是FrameworkServlet提供的模板方法，由子类DispatcherServlet来实现的。

```java
/**
  * This implementation calls {@link #initStrategies}.
  */
@Override
protected void onRefresh(ApplicationContext context) {
  initStrategies(context);
}
```

刷新主要是调用initStrategies(context)方法对DispatcherServlet中的组件进行初始化，这些组件就是在SpringMVC请求流程中包的主要组件。

```java
/**
  * Initialize the strategy objects that this servlet uses.
  * <p>May be overridden in subclasses in order to initialize further strategy objects.
  */
protected void initStrategies(ApplicationContext context) {
  initMultipartResolver(context);
  initLocaleResolver(context);
  initThemeResolver(context);

  // 主要看如下三个方法
  initHandlerMappings(context);
  initHandlerAdapters(context);
  initHandlerExceptionResolvers(context);

  initRequestToViewNameTranslator(context);
  initViewResolvers(context);
  initFlashMapManager(context);
}
```

### [#](#inithanlderxxx) initHanlderxxx

我们主要看initHandlerXXX相关的方法，它们之间的关系可以看SpringMVC的请求流程：

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723260289-fd30d949e6256f58ce993f779a367d60-1723260470322-5.png)

1.  HandlerMapping是映射处理器
2.  HandlerAdpter是**处理适配器**，它用来找到你的Controller中的处理方法
3.  HandlerExceptionResolver是当遇到处理异常时的异常解析器

initHandlerMapping方法如下，无非就是获取按照优先级排序后的HanlderMappings, 将来匹配时按照优先级最高的HanderMapping进行处理。





# [#](#spring%E8%BF%9B%E9%98%B6-springmvc%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E4%B9%8Bdispatcherservlet%E5%A4%84%E7%90%86%E8%AF%B7%E6%B1%82%E7%9A%84%E8%BF%87%E7%A8%8B) Spring进阶 - SpringMVC实现原理之DispatcherServlet处理请求的过程

> 前文我们有了IOC的源码基础以及SpringMVC的基础，我们便可以进一步深入理解SpringMVC主要实现原理，包含DispatcherServlet的初始化过程和DispatcherServlet处理请求的过程的源码解析。本文是第二篇：DispatcherServlet处理请求的过程的源码解析。@pdai

*   [Spring进阶 - SpringMVC实现原理之DispatcherServlet处理请求的过程](#spring%E8%BF%9B%E9%98%B6---springmvc%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E4%B9%8Bdispatcherservlet%E5%A4%84%E7%90%86%E8%AF%B7%E6%B1%82%E7%9A%84%E8%BF%87%E7%A8%8B)
    *   [DispatcherServlet处理请求的过程？](#dispatcherservlet%E5%A4%84%E7%90%86%E8%AF%B7%E6%B1%82%E7%9A%84%E8%BF%87%E7%A8%8B)
        *   [回顾整理处理流程](#%E5%9B%9E%E9%A1%BE%E6%95%B4%E7%90%86%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B)
        *   [doGet入口](#doget%E5%85%A5%E5%8F%A3)
        *   [请求分发](#%E8%AF%B7%E6%B1%82%E5%88%86%E5%8F%91)
        *   [映射和适配器处理](#%E6%98%A0%E5%B0%84%E5%92%8C%E9%80%82%E9%85%8D%E5%99%A8%E5%A4%84%E7%90%86)
        *   [视图渲染](#%E8%A7%86%E5%9B%BE%E6%B8%B2%E6%9F%93)

## [#](#dispatcherservlet%E5%A4%84%E7%90%86%E8%AF%B7%E6%B1%82%E7%9A%84%E8%BF%87%E7%A8%8B) DispatcherServlet处理请求的过程？

> 一个请求发出，经过DispatcherServlet进行了什么样的处理，最后将内容返回的呢？

### [#](#%E5%9B%9E%E9%A1%BE%E6%95%B4%E7%90%86%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B) 回顾整理处理流程

首先让我们整体看一下Spring Web MVC 处理请求的流程：

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723260571-91907ed8ef56da64b7bc0ce99073a0e1-1723260685376-15.png)

**核心架构的具体流程步骤**如下：

1.  **首先用户发送请求——>DispatcherServlet**，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行 处理，作为统一访问点，进行全局的流程控制；
2.  **DispatcherServlet——>HandlerMapping**， HandlerMapping 将会把请求映射为 HandlerExecutionChain 对象（包含一 个Handler 处理器（页面控制器）对象、多个HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新的映射策略；
3.  **DispatcherServlet——>HandlerAdapter**，HandlerAdapter 将会把处理器包装为适配器，从而支持多种类型的处理器， 即适配器设计模式的应用，方便扩展更多类型的处理器；
4.  **HandlerAdapter——>处理器功能处理方法的调用**，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处 理方法，完成功能处理；并返回一个ModelAndView 对象（包含模型数据、逻辑视图名）；
5.  **ModelAndView 的逻辑视图名——> ViewResolver**，ViewResolver 将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；
6.  **View——>渲染**，View 会根据传进来的Model 模型数据进行渲染，此处的Model 实际是一个Map 数据结构，因此 很容易支持其他视图技术；
7.  **返回控制权给DispatcherServlet**，由DispatcherServlet 返回响应给用户，到此一个流程结束。

### [#](#doget%E5%85%A5%E5%8F%A3) doGet入口

> 我们以上个demo中这个GET请求为例，请求URL是http://localhost:8080/011\_spring\_framework\_demo\_springmvc\_war\_exploded/user

我们知道servlet处理get请求是doGet方法，所以我们去找DispatcherServlet类结构中的doGet方法。

```java
@Override
protected final void doGet(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

  processRequest(request, response);
}
```

processRequest处理请求的方法如下：

```java
/**
  * Process this request, publishing an event regardless of the outcome.
  * <p>The actual event handling is performed by the abstract
  * {@link #doService} template method.
  */
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

  // 计算处理请求的时间
  long startTime = System.currentTimeMillis();
  Throwable failureCause = null;

  LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
  LocaleContext localeContext = buildLocaleContext(request);

  RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
  ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

  WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
  asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

  // 初始化context
  initContextHolders(request, localeContext, requestAttributes);

  try {
    // 看这里
    doService(request, response);
  }
  catch (ServletException | IOException ex) {
    failureCause = ex;
    throw ex;
  }
  catch (Throwable ex) {
    failureCause = ex;
    throw new NestedServletException("Request processing failed", ex);
  }

  finally {
    // 重置context
    resetContextHolders(request, previousLocaleContext, previousAttributes);
    if (requestAttributes != null) {
      requestAttributes.requestCompleted();
    }
    logResult(request, response, failureCause, asyncManager);
    publishRequestHandledEvent(request, response, startTime, failureCause);
  }
}
```

本质上就是调用doService方法，由DispatchServlet类实现

```java
/**
  * Exposes the DispatcherServlet-specific request attributes and delegates to {@link #doDispatch}
  * for the actual dispatching.
  */
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
  logRequest(request);

  // 保存下请求之前的参数.
  Map<String, Object> attributesSnapshot = null;
  if (WebUtils.isIncludeRequest(request)) {
    attributesSnapshot = new HashMap<>();
    Enumeration<?> attrNames = request.getAttributeNames();
    while (attrNames.hasMoreElements()) {
      String attrName = (String) attrNames.nextElement();
      if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
        attributesSnapshot.put(attrName, request.getAttribute(attrName));
      }
    }
  }

  // 方便后续 handlers 和 view 要使用它们.
  request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
  request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
  request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
  request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

  if (this.flashMapManager != null) {
    FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
    if (inputFlashMap != null) {
      request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
    }
    request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
    request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
  }

  RequestPath previousRequestPath = null;
  if (this.parseRequestPath) {
    previousRequestPath = (RequestPath) request.getAttribute(ServletRequestPathUtils.PATH_ATTRIBUTE);
    ServletRequestPathUtils.parseAndCache(request);
  }

  try {
    // 看这里，终于将这个请求分发出去了
    doDispatch(request, response);
  }
  finally {
    if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
      // Restore the original attribute snapshot, in case of an include.
      if (attributesSnapshot != null) {
        restoreAttributesAfterInclude(request, attributesSnapshot);
      }
    }
    if (this.parseRequestPath) {
      ServletRequestPathUtils.setParsedRequestPath(previousRequestPath, request);
    }
  }
}
```

### [#](#%E8%AF%B7%E6%B1%82%E5%88%86%E5%8F%91) 请求分发

doDispatch方法是真正处理请求的核心方法

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
  HttpServletRequest processedRequest = request;
  HandlerExecutionChain mappedHandler = null;
  boolean multipartRequestParsed = false;

  WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

  try {
    ModelAndView mv = null;
    Exception dispatchException = null;

    try {
      // 判断是不是文件上传类型的request
      processedRequest = checkMultipart(request);
      multipartRequestParsed = (processedRequest != request);

      // 根据request获取匹配的handler.
      mappedHandler = getHandler(processedRequest);
      if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return;
      }

      // 根据handler获取匹配的handlerAdapter
      HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

      // 如果handler支持last-modified头处理
      String method = request.getMethod();
      boolean isGet = HttpMethod.GET.matches(method);
      if (isGet || HttpMethod.HEAD.matches(method)) {
        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
        if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
          return;
        }
      }

      if (!mappedHandler.applyPreHandle(processedRequest, response)) {
        return;
      }

      // 真正handle处理，并返回modelAndView
      mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

      if (asyncManager.isConcurrentHandlingStarted()) {
        return;
      }

      // 通过视图的prefix和postfix获取完整的视图名
      applyDefaultViewName(processedRequest, mv);

      // 应用后置的拦截器
      mappedHandler.applyPostHandle(processedRequest, response, mv);
    }
    catch (Exception ex) {
      dispatchException = ex;
    }
    catch (Throwable err) {
      // As of 4.3, we're processing Errors thrown from handler methods as well,
      // making them available for @ExceptionHandler methods and other scenarios.
      dispatchException = new NestedServletException("Handler dispatch failed", err);
    }

    // 处理handler处理的结果，显然就是对ModelAndView 或者 出现的Excpetion处理
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
  }
  catch (Exception ex) {
    triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
  }
  catch (Throwable err) {
    triggerAfterCompletion(processedRequest, response, mappedHandler,
        new NestedServletException("Handler processing failed", err));
  }
  finally {
    if (asyncManager.isConcurrentHandlingStarted()) {
      // Instead of postHandle and afterCompletion
      if (mappedHandler != null) {
        mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
      }
    }
    else {
      // Clean up any resources used by a multipart request.
      if (multipartRequestParsed) {
        cleanupMultipart(processedRequest);
      }
    }
  }
}
```

### [#](#%E6%98%A0%E5%B0%84%E5%92%8C%E9%80%82%E9%85%8D%E5%99%A8%E5%A4%84%E7%90%86) 映射和适配器处理

对于真正的handle方法，我们看下其处理流程

```java
/**
  * This implementation expects the handler to be an {@link HandlerMethod}.
  */
@Override
@Nullable
public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
    throws Exception {

  return handleInternal(request, response, (HandlerMethod) handler);
}
```

交给handleInternal方法处理，以RequestMappingHandlerAdapter这个HandlerAdapter中的处理方法为例

```java
@Override
protected ModelAndView handleInternal(HttpServletRequest request,
    HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

  ModelAndView mav;
  checkRequest(request);

  // Execute invokeHandlerMethod in synchronized block if required.
  if (this.synchronizeOnSession) {
    HttpSession session = request.getSession(false);
    if (session != null) {
      Object mutex = WebUtils.getSessionMutex(session);
      synchronized (mutex) {
        mav = invokeHandlerMethod(request, response, handlerMethod);
      }
    }
    else {
      // No HttpSession available -> no mutex necessary
      mav = invokeHandlerMethod(request, response, handlerMethod);
    }
  }
  else {
    // No synchronization on session demanded at all...
    mav = invokeHandlerMethod(request, response, handlerMethod);
  }

  if (!response.containsHeader(HEADER_CACHE_CONTROL)) {
    if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
      applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
    }
    else {
      prepareResponse(response);
    }
  }

  return mav;
}
```

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723260571-03937bc982bf1cafd8146971cfe9504a-1723260685375-13.png)

然后执行invokeHandlerMethod这个方法，用来对RequestMapping（usercontroller中的list方法）进行处理

```java
/**
  * Invoke the {@link RequestMapping} handler method preparing a {@link ModelAndView}
  * if view resolution is required.
  * @since 4.2
  * @see #createInvocableHandlerMethod(HandlerMethod)
  */
@Nullable
protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
    HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

  ServletWebRequest webRequest = new ServletWebRequest(request, response);
  try {
    
    WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
    ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

    // 重要：设置handler(controller#list)方法上的参数，返回值处理，绑定databinder等
    ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
    if (this.argumentResolvers != null) {
      invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
    }
    if (this.returnValueHandlers != null) {
      invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
    }
    invocableMethod.setDataBinderFactory(binderFactory);
    invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);

    ModelAndViewContainer mavContainer = new ModelAndViewContainer();
    mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
    modelFactory.initModel(webRequest, mavContainer, invocableMethod);
    mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);

    
    AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
    asyncWebRequest.setTimeout(this.asyncRequestTimeout);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.setTaskExecutor(this.taskExecutor);
    asyncManager.setAsyncWebRequest(asyncWebRequest);
    asyncManager.registerCallableInterceptors(this.callableInterceptors);
    asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

    if (asyncManager.hasConcurrentResult()) {
      Object result = asyncManager.getConcurrentResult();
      mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
      asyncManager.clearConcurrentResult();
      LogFormatUtils.traceDebug(logger, traceOn -> {
        String formatted = LogFormatUtils.formatValue(result, !traceOn);
        return "Resume with async result [" + formatted + "]";
      });
      invocableMethod = invocableMethod.wrapConcurrentResult(result);
    }

    // 执行controller中方法
    invocableMethod.invokeAndHandle(webRequest, mavContainer);
    if (asyncManager.isConcurrentHandlingStarted()) {
      return null;
    }

    return getModelAndView(mavContainer, modelFactory, webRequest);
  }
  finally {
    webRequest.requestCompleted();
  }
}
```

invokeAndHandle交给UserController中具体执行list方法执行

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723260571-eb4a2308ef1c53b0abf6f65b259272eb-1723260685376-14.png)

后续invoke执行的方法，直接看整个请求流程的调用链即可

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723260571-08912a5451d3b40be248d82d303c0efb-1723260685376-17.png)

执行后获得视图和Model

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/SpringMVC请求流程设计_Img/1723260571-f343e34248bb185597397f951e0ed1f6-1723260685376-16.png)

### [#](#%E8%A7%86%E5%9B%BE%E6%B8%B2%E6%9F%93) 视图渲染

接下来继续执行processDispatchResult方法，对视图和model（如果有异常则对异常处理）进行处理（显然就是渲染页面了）

```java
/**
  * Handle the result of handler selection and handler invocation, which is
  * either a ModelAndView or an Exception to be resolved to a ModelAndView.
  */
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
    @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
    @Nullable Exception exception) throws Exception {

  boolean errorView = false;

  // 如果处理过程有异常，则异常处理
  if (exception != null) {
    if (exception instanceof ModelAndViewDefiningException) {
      logger.debug("ModelAndViewDefiningException encountered", exception);
      mv = ((ModelAndViewDefiningException) exception).getModelAndView();
    }
    else {
      Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
      mv = processHandlerException(request, response, handler, exception);
      errorView = (mv != null);
    }
  }

  // 是否需要渲染视图
  if (mv != null && !mv.wasCleared()) {
    render(mv, request, response); // 渲染视图
    if (errorView) {
      WebUtils.clearErrorRequestAttributes(request);
    }
  }
  else {
    if (logger.isTraceEnabled()) {
      logger.trace("No view rendering, null ModelAndView returned.");
    }
  }

  if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
    // Concurrent handling started during a forward
    return;
  }

  if (mappedHandler != null) {
    // Exception (if any) is already handled..
    mappedHandler.triggerAfterCompletion(request, response, null);
  }
}
```

接下来显然就是渲染视图了, spring在initStrategies方法中初始化的组件（LocaleResovler等）就派上用场了。

```java
/**
  * Render the given ModelAndView.
  * <p>This is the last stage in handling a request. It may involve resolving the view by name.
  * @param mv the ModelAndView to render
  * @param request current HTTP servlet request
  * @param response current HTTP servlet response
  * @throws ServletException if view is missing or cannot be resolved
  * @throws Exception if there's a problem rendering the view
  */
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
  // Determine locale for request and apply it to the response.
  Locale locale =
      (this.localeResolver != null ? this.localeResolver.resolveLocale(request) : request.getLocale());
  response.setLocale(locale);

  View view;
  String viewName = mv.getViewName();
  if (viewName != null) {
    // We need to resolve the view name.
    view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
    if (view == null) {
      throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
          "' in servlet with name '" + getServletName() + "'");
    }
  }
  else {
    // No need to lookup: the ModelAndView object contains the actual View object.
    view = mv.getView();
    if (view == null) {
      throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a " +
          "View object in servlet with name '" + getServletName() + "'");
    }
  }

  // Delegate to the View object for rendering.
  if (logger.isTraceEnabled()) {
    logger.trace("Rendering view [" + view + "] ");
  }
  try {
    if (mv.getStatus() != null) {
      response.setStatus(mv.getStatus().value());
    }
    view.render(mv.getModelInternal(), request, response);
  }
  catch (Exception ex) {
    if (logger.isDebugEnabled()) {
      logger.debug("Error rendering view [" + view + "]", ex);
    }
    throw ex;
  }
}
```

后续就是通过viewResolver进行解析了，这里就不再继续看代码了，上述流程基本上够帮助你构建相关的认知了。

最后无非是返回控制权给DispatcherServlet，由DispatcherServlet 返回响应给用户。

最后的最后我们看下请求的日志：

```bash
21:45:53.390 [http-nio-8080-exec-6] DEBUG org.springframework.web.servlet.DispatcherServlet - GET "/011_spring_framework_demo_springmvc_war_exploded/user", parameters={}
21:45:53.400 [http-nio-8080-exec-6] DEBUG org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping - Mapped to tech.pdai.springframework.springmvc.controller.UserController#list(HttpServletRequest, HttpServletResponse)
22:51:14.504 [http-nio-8080-exec-6] DEBUG org.springframework.web.servlet.view.JstlView - View name 'userList', model {dateTime=Fri Apr 22 21:45:53 CST 2022, userList=[tech.pdai.springframework.springmvc.entity.User@7b8c8dc]}
22:51:14.550 [http-nio-8080-exec-6] DEBUG org.springframework.web.servlet.view.JstlView - Forwarding to [/WEB-INF/views/userList.jsp]
22:51:44.395 [http-nio-8080-exec-6] DEBUG org.springframework.web.servlet.DispatcherServlet - Completed 200 OK
```
