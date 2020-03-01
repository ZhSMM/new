# SpringMVC传参

## 前置准备

### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <url-pattern>*.action</url-pattern>
    </servlet-mapping>
</web-app>
```

### classpath:spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="top.songfang.controller"/>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="suffix" value=".jsp"/>
        <property name="prefix" value="/views/"/>
    </bean>
</beans>
```

### Student类

```java
// Student
package top.songfang.entity;

public class Student {
    private int id;
    private String name;
    private Address address;

    public Student() {
    }

    public Student(int id, String name, Address address) {
        this.id = id;
        this.name = name;
        this.address = address;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", address=" + address +
                '}';
    }
    -----------setter/getter---------
}

// Address类
package top.songfang.entity;

public class Address {
    private String schoolAddress;
    private String homeAddress;

    public Address() {
    }

    public Address(String schoolAddress, String homeAddress) {
        this.schoolAddress = schoolAddress;
        this.homeAddress = homeAddress;
    }
    @Override
    public String toString() {
        return "Address{" +
                "schoolAddress='" + schoolAddress + '\'' +
                ", homeAddress='" + homeAddress + '\'' +
                '}';
    }
    ------------setter/getter---------
}
```

### WEB-INF/views/direct.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: 张松
  Date: 2020/3/1
  Time: 14:15
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="restGet/10/mjl.action" method="get">
    <input type="submit" value="restGet">
</form>
<form action="restPost/10/mjkd.action" method="post">
    <input type="submit" value="restPost">
</form>
<form action="restPut/10/mjkd.action" method="post">
    <input type="hidden" name="_method" value="put">
    <input type="submit" value="restPut">
</form>
<form action="restDelete/10/mjkd.action" method="post">
    <input type="hidden" name="_method" value="delete">
    <input type="submit" value="restDelete">
</form>
<form
</body>
</html>
```

### WEB-INF/views/rest.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: 张松
  Date: 2020/3/1
  Time: 10:46
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
studentID:${requestScope.studentID} <br/>
studentName:${requestScope.studentName}<br/>
requestMethod:${requestScope.requestMethod}
</body>
</html>
```

### WEB-INF/views/student.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: 张松
  Date: 2020/2/29
  Time: 23:50
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<table border="1" align="center">
    <tr>
        <th>student ID:</th>
        <th>${requestScope.student0.id}</th>
    </tr>
    <tr>
        <td>student name:</td>
        <td>${requestScope.student0.name}</td>
    </tr>
    <tr>
        <td>homeAddress:</td>
        <td>${requestScope.student0.address.homeAddress}</td>
    </tr>
    <tr>
        <td>student name:</td>
        <td>${requestScope.student0.address.schoolAddress}</td>
    </tr>
</table>
<table border="1" align="center">
    <tr>
        <th>student ID:</th>
        <th>${sessionScope.student0.id}</th>
    </tr>
    <tr>
        <td>student name:</td>
        <td>${sessionScope.student0.name}</td>
    </tr>
    <tr>
        <td>homeAddress:</td>
        <td>${sessionScope.student0.address.homeAddress}</td>
    </tr>
    <tr>
        <td>student name:</td>
        <td>${sessionScope.student0.address.schoolAddress}</td>
    </tr>
</table>
</body>
</html>
```

## Rest风格传参

### RestController

```java
package top.songfang.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import java.util.Map;

@Controller
@RequestMapping("/views/")
public class RestfulController {

    @RequestMapping(value="restGet/{id}/{name}.action",method = RequestMethod.GET)
    public String restGet(@PathVariable(value = "id")Integer id,
                          @PathVariable(value = "name")String name, 
                          Map<String,Object> map,
                          HttpServletRequest request){
        map.put("studentID",id);
        map.put("studentName",name);
        map.put("requestMethod",request.getMethod());
        return "rest";
    }
    @RequestMapping(value = "restPost/{id}/{name}.action",
                    method = RequestMethod.POST)
    public String restPost(@PathVariable(value = "id")Integer id,
                           @PathVariable(value = "name")String name,
                           ModelMap map,
                           HttpServletRequest request){
        map.put("studentID",id);
        map.put("studentName",name);
        map.put("requestMethod",request.getMethod());
        return "rest";
    }
    
    @RequestMapping(value = "restPut/{id}/{name}.action",
                    method = RequestMethod.PUT)
    public String restPut(@PathVariable(value = "id")Integer id,
                           @PathVariable(value = "name")String name,
                           HttpServletRequest request){
        return "redirect:/views/restToPut/"+id+"/"+name
            +".action?request="+request.getMethod();
    }
    
    @RequestMapping(value = "restDelete/{id}/{name}.action",method = RequestMethod.DELETE)
    public String restDelete(@PathVariable(value = "id")Integer id,
                             @PathVariable(value = "name")String name,
                             HttpServletRequest servlet){
        return "redirect:/views/restToDelete/"+id+"/"+name
            +".action?request="+servlet.getMethod();
    }
    @RequestMapping(value="restToDelete/{id}/{name}.action",
                    params ="request",method = RequestMethod.GET)
    public ModelAndView restToDelete(@PathVariable(value = "id")Integer id,
                                     @PathVariable(value = "name")String name,
                                     @RequestParam(value = "request")String request){
        ModelAndView mv=new ModelAndView("rest");
        mv.addObject("studentID",id);
        mv.addObject("studentName",name);
        mv.addObject("requestMethod",request);
        return mv;
    }
    @RequestMapping(value = "restToPut/{id}/{name}.action",
                    params = "request",method = RequestMethod.GET)
    public String restToPut(@PathVariable(value = "id")Integer id,
                            @PathVariable(value = "name")String name, 
                            @RequestParam("request")String request,
                            Model model){
        model.addAttribute("studentID",id);
        model.addAttribute("studentName",name);
        model.addAttribute("requestMethod",request);
        return "rest";
    }
}
```

## @RequestParameter传参

```java
package top.songfang.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;

import org.springframework.web.servlet.ModelAndView;
import top.songfang.entity.Student;

/**
 *  SpringMVC处理各种参数流程/逻辑：
 *    请求：  前端发请求a -> @RequestMapping("a")
 *    处理请求中的参数： public String aa(@RequestParam("a")String str) 各种变量注解
 *
 *  使用对象（实体类接受参数）
 *
 */

@Controller
public class RestController {
    // rest风格通过@PathVariable获得值
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
    // @RequestParam("name")等价于request.getParameter("name"),接受从前端的传值
    //   required=false 说明这个参数可以不配
    //   defaultValue = "18" 配置默认值
    @RequestMapping(value = "get.action",method = RequestMethod.GET)
    public String getName(@RequestParam("name") String name,@RequestParam(value="age",required = false,defaultValue = "18")Integer age){
        System.out.println(name+age);
        return "success";
    }
    // @RequestHeader("key")String al:获取头信息相应key的值
    @RequestMapping(value = "header.action",method = RequestMethod.GET)
    public String getRequest(@RequestHeader("Accept-Language")String str){
        System.out.println(str);
        return "success";
    }
    // @CookieValue
    // 前置知识：服务端接受到客户端第一次请求时，会给该客户端分配一个session（该session包含一个sessionID），
    //   并且服务端会在第一次响应客户端时，会将sessionID赋值给JSESSIONID并传递给客户端。
    @RequestMapping(value = "cookie.action",method = RequestMethod.GET)
    public String getCookie(@CookieValue("JSESSIONID")String jsessionid){
        System.out.println(jsessionid);
        return "success";
    }
    // 自定义对象传入
    // 1. name属性的值必须和对象的属性名一一对应；
    // 2. 支持级联传递，及一个对象可以包含另一个对象，如本例address.homeAddress和address.SchoolAddress
    @RequestMapping("student.action")
    public String getStudent(Student student){
        System.out.println(student);
        return "success";
    }

    // 在SpringMVC中使用原生态的Servlet API
    // 只需要将参数写入即可
    // 在这里有一个小的坑，需要引入Tomcat的依赖，idea中的操作方法：
    //     File->Project Structure->Modules->右侧的项目名-> 选择dependence ->右侧+号-> Library
    @RequestMapping("servlet.action")
    public String getServlet(HttpServletRequest request){
        System.out.println(request.getServletPath());
        return "success";
    }
}

```

