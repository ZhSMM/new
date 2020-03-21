# XML装配Bean

需要创建一个xml文件，且要以`<beans>`元素为根。

声明简单的`<bean>`：`<bean id="id" class="类的全路径" />`

最简单的spring XML配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
       xsi:schemaLocation="http://www.springframework.org/schema/beans">
	<!-- 具体配置信息 -->
</beans>
```

Students类和Pen类：

```java
// Pen类
public class Pen {
    private String color;
    private final Date date=new Date();
}
// Students类
public class Student {
    private String name;
    private List<Pen> pens;
    private List<String> hobby;
    private Set<String> subject;
    
    private Map<String,Integer> prices;
    // 空构造不能少
    public Student(){}
    // 省略了非空构造方法以及setter/getter
}
```

### 构造方法注入（需要有参构造）

1. 原始版本

   ```xml
   <bean id="blackPen" class="learn.spring.xml.entity.Pen">
       <constructor-arg name="color" value="black"/>
   </bean>
   <bean id="whitePen" class="learn.spring.xml.entity.Pen">
       <constructor-arg name="color" value="white"/>
   </bean>
   <bean id="redPen" class="learn.spring.xml.entity.Pen">
       <constructor-arg name="color" value="red"/>
   </bean>
   
   <util:list id="hobbyList">
       <value>打篮球</value>
       <value>打网球</value>
       <value>看剧</value>
   </util:list>
   
   <bean id="student" class="learn.spring.xml.entity.Student">
       <constructor-arg name="name" value="张三"/>
       <constructor-arg name="pens">
           <list>
               <ref bean="blackPen"/>
               <ref bean="redPen"/>
               <ref bean="whitePen"/>
           </list>
       </constructor-arg>
       
       <constructor-arg name="hobby" ref="hobbyList" />
       
       <constructor-arg name="subject">
           <set>
               <value>语文</value>
               <value>数学</value>
           </set>
       </constructor-arg>
       <constructor-arg name="prices">
           <map>
               <entry value="18" key="橘子"/>
               <entry value="20" key="栗子"/>
               <entry value="38" key="黄瓜"/>
           </map>
       </constructor-arg>
   </bean>
   ```

2. 引入c命名空间：

   ```xml
   <!--
   	在beans标签中引入c命名空间：
       xmlns:c="http://www.springframework.org/schema/c"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans/spring-beans.xsd
   
   	在beans标签中引入util命名空间：
   	xmlns:util="http://www.springframework.org/schema/util"
   	xsi:schemaLocation="
   		http://www.springframework.org/schema/util 		
   		http://www.springframework.org/schema/util/spring-util.xsd"
   -->
   
   <bean id="blackPen" class="learn.spring.xml.entity.Pen" c:color="black"/>
   <bean id="whitePen" class="learn.spring.xml.entity.Pen" c:color="white"/>
   <bean id="redPen" class="learn.spring.xml.entity.Pen" c:color="red"/>
   
   <util:list id="pensList">
       <ref bean="blackPen"/>
       <ref bean="whitePen"/>
       <ref bean="redPen"/>
   </util:list>
   <util:list id="hobbyList">
       <value>打篮球</value>
       <value>打网球</value>
       <value>看剧</value>
   </util:list>
   <util:set id="set">
       <value>语文</value>
       <value>数学</value>
   </util:set>
   <util:map id="map">
       <entry value="18" key="橘子"/>
       <entry value="20" key="栗子"/>
       <entry value="38" key="黄瓜"/>
   </util:map>
   <bean id="student" class="learn.spring.xml.entity.Student"
         c:name="张三"
         c:pens-ref="pensList"
         c:hobby-ref="hobbyList"
         c:subject-ref="set"
         c:prices-ref="map"/>
   ```

### Set方法注入

1. 原始方法注入

   ```xml
   <bean id="blackPen" class="learn.spring.xml.entity.Pen">
       <property name="color" value="black"/>
   </bean>
   <bean id="whitePen" class="learn.spring.xml.entity.Pen">
       <property name="color" value="white"/>
   </bean>
   <bean id="redPen" class="learn.spring.xml.entity.Pen">
       <property name="color" value="red"/>
   </bean>
   <bean id="student" class="learn.spring.xml.entity.Student">
       <property name="name" value="张三"/>
       <property name="pens">
           <list>
               <ref bean="blackPen"/>
               <ref bean="redPen"/>
               <ref bean="whitePen"/>
           </list>
       </property>
       <property name="hobby">
           <list>
               <value>打篮球</value>
               <value>打网球</value>
               <value>看剧</value>
           </list>
       </property>
       <property name="subject">
           <set>
               <value>语文</value>
               <value>数学</value>
           </set>
       </property>
       <property name="prices">
           <map>
               <entry value="18" key="橘子"/>
               <entry value="20" key="栗子"/>
               <entry value="38" key="黄瓜"/>
           </map>
       </property>
   </bean>
   ```

2. 引入p命名空间

   ```xml
   <!--
   	在beans标签中添加p命名空间：
   	xmlns:p="http://www.springframework.org/schema/p"
   	xsi:schemaLocation="http://www.springframework.org/schema/beans/spring-beans.xsd
   -->
   <bean id="blackPen" class="learn.spring.xml.entity.Pen" p:color="black"/>
   <bean id="whitePen" class="learn.spring.xml.entity.Pen" p:color="white"/>
   <bean id="redPen" class="learn.spring.xml.entity.Pen" p:color="red"/>
   
   <util:list id="pensList">
       <ref bean="blackPen"/>
       <ref bean="whitePen"/>
       <ref bean="redPen"/>
   </util:list>
   <util:list id="hobbyList">
       <value>打篮球</value>
       <value>打网球</value>
       <value>看剧</value>
   </util:list>
   <util:set id="set">
       <value>语文</value>
       <value>数学</value>
   </util:set>
   <util:map id="map">
       <entry value="18" key="橘子"/>
       <entry value="20" key="栗子"/>
       <entry value="38" key="黄瓜"/>
   </util:map>
   <bean id="student" class="learn.spring.xml.entity.Student"
         p:name="张三"
         p:pens-ref="pensList"
         p:hobby-ref="hobbyList"
         p:subject-ref="set"
         p:prices-ref="map"/>
   ```

### c命名空间与p命名空间命名规则

- c:[构造器参数名：自定义]-ref="[要注入的bean的id]";
- c:_0-ref="[要注入的bean的id]";   _0表示参数索引
- c:_-ref="[要注入的bean的id]";    _占位符

### util-命名空间的部分成员

|        元素        |                        描述                         |
| :----------------: | :-------------------------------------------------: |
| `<util:constant>`  |  引用某个类型的 public static域，并将其暴露为bean   |
|     util:list      | 创建一个 java.util.List类型的bean，其中包含值或引用 |
|      util:map      | 创建一个 java.util.Map类型的bean，其中包含值或引用  |
|  util:properties   |       创建一个 java.util.Properties类型的bean       |
| util:property-path | 引用一个 bean的属性（或内嵌属性），并将其暴露为bean |
|      util:set      | 创建一个 java.util.Set类型的bean，其中包含值或引用  |