# SpringMVC初步

## 小示例

### 配置web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">


    <!-- 配置SpringMVC前端控制器DispatcherServlet -->
    <servlet>
        <servlet-name>springDispatcherServlet</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        
        <!--
            1. 初始化参数，包括springmvc.xml的位置
            2. 未设置，将springmvc放入默认路径即可：
                1. 命名格式为 servlet-name-servlet.xml，
					如本例：springDispatcherServlet-servlet.xml
                2. 默认目录：WEB-INF目录下
        -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!-- tomcat启动时最先加载 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>springDispatcherServlet</servlet-name>
        <!--
            1. 精准拦截：/开头，后面为具体路径，如 /index.jsp
            2. 最长路径：/开头，/*结束，如 /index/* , /*
            3. 扩展名： 以*.开头，以扩展名结束，如 *.action
            4. 默认，全部拦截 /
        -->
        <url-pattern>*.action</url-pattern>
    </servlet-mapping>

</web-app>
```

### 配置springmvc.xml

这里springmvc.xml被我放到了classpath目录下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
    <!-- 扫描有注解的包 -->
    <context:component-scan base-package="top.songfang.controller"/>
    
    <!-- 配置视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    
</beans>
```

### 简单的Controller

Controller又叫Handler、Servlet

```java
package top.songfang.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

// 注解或配置
@Controller
@RequestMapping("hello/")
public class SpringMVCHandler {

    /**
     * 请求跳转方式默认为请求转发的跳转方式
     * RequestMapping参数：
     *   value：请求的url地址
     *   method:指定请求方式 RequestMethod.POST、GET、PUT、DELETE
     *   params:请求必须携带的参数，用""包裹，如"name",支持类似"name!=23",
     *     多参数用，分隔，如"name=zh","age!=12"
     *   headers:约定请求头信息
     *
     * ant风格：
     *   ？单个字符匹配，*任意字符（0-n），**任意目录
     *
     * Rest风格：
     *   浏览器支持： Get：查 、Post：改
     *   浏览器不支持：Delete:删、Put：增
     *
     * 通过过滤器实现Delete、Put支持（SpringMVC提供）
     *   条件：
     *     1. <input type="hidden" name="_method" value="delete/put">
     *     2. 请求方式为post
     *   配置filter
     */
    @RequestMapping(value = "welcome.action",method = RequestMethod.POST, params = {"name"})
    public String welcome(){
        // 在上面配置了视图解析器的前后缀，会被拼接成/views/welcome.jsp
        return "welcome";  
    }
}
```

### index.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: 张松
  Date: 2020/2/29
  Time: 10:25
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
  <!-- jsp里面的连接不支持前面加/,如/hello/welcome.action会报错，只能写成hello/welcome.action -->
  <a href="hello/welcome.action">欢迎</a>
  </body>
</html>
```

### welcome.jsp

该文件在WEB-INF/views/目录下面：

```jsp
<%--
  Created by IntelliJ IDEA.
  User: 张松
  Date: 2020/2/29
  Time: 10:51
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>Success!</h1>
</body>
</html>
```



