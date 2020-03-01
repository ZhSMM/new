# SpringMVC实现简单Rest风格

## Web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>springDispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springDispatcherServlet</servlet-name>
        <url-pattern>*.action</url-pattern>
    </servlet-mapping>

    <filter>
        <filter-name>hiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>hiddenHttpMethodFilter</filter-name>
        <url-pattern>/</url-pattern>
    </filter-mapping>
</web-app>
```

## spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="top.songfang.controller"/>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

## index.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: 张松
  Date: 2020/2/29
  Time: 17:51
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>Index</title>
  </head>
  <form action="rest/delete.action" method="post">
    <input type="hidden" name="_method" value="delete">
    <input name="name">
    <input type="submit" value="delete">
  </form>
  <form action="rest/put.action" method="put">
    <input type="hidden" name="_method" value="put">
    <input name="name">
    <input type="submit" value="put">
  </form>
  <form action="rest/post.action" method="post">
    <input name="name">
    <input type="submit" value="post">
  </form>
  <form action="rest/get.action" method="get">
    <input name="name">
    <input type="submit" value="get">
  </form>
  </body>
</html>
```

## success.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: 张松
  Date: 2020/2/29
  Time: 17:58
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1 align="center">跳转成功！</h1>
</body>
</html>
```

## Controller

```java
package top.songfang.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
public class RestController {
    /**
    * SpringMVC支持请求的url一样，请求方式不同来区分请求；以实现rest风格的增删改查
    */
    @RequestMapping(value = "/rest/{name}",method = RequestMethod.POST)
    public String post(@PathVariable("name")String name){
        System.out.println(name);
        return "success";
    }
    @RequestMapping(value = "/rest/{name}",method = RequestMethod.GET)
    public String get(@PathVariable("name")String name){
        System.out.println(name);
        return "success";
    }
    @RequestMapping(value = "/rest/{name}",method = RequestMethod.PUT)
    public String put(@PathVariable("name")String name){
        System.out.println(name);
        return "success";
    }
    @RequestMapping(value = "/rest/{name}",method = RequestMethod.DELETE)
    public String delete(@PathVariable("name")String name){
        System.out.println(name);
        return "success";
    }
}

```



