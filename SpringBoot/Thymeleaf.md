# Thymeleaf

Thymeleaf是一种Java XML / XHTML / HTML5模板引擎，可以在Web和非Web环境中使用。它更适合在基于MVC的Web应用程序的视图层提供XHTML / HTML5，但即使在脱机环境中，它也可以处理任何XML文件。它提供了完整的Spring Framework集成。

## maven配置

```xml
<!--thymeleaf模板-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## 常用配置

```properties
# 启用缓存:建议生产开启
spring.thymeleaf.cache=false
# 建议模版是否存在
spring.thymeleaf.check-template-location=true
# Content-Type 值
spring.thymeleaf.servlet.content-type=text/html
# 是否启用
spring.thymeleaf.enabled=true
# 模版编码
spring.thymeleaf.encoding=utf-8
# 应该从解析中排除的视图名称列表（用逗号分隔）
spring.thymeleaf.excluded-view-names=
# 模版模式
spring.thymeleaf.mode=HTML5
# 模版存放路径
spring.thymeleaf.prefix=classpath:/templates/
# 模版后缀
spring.thymeleaf.suffix=.html
```

| 配置项                                |  类型  |        默认值         | 建议值 | 说明                                 |
| :------------------------------------ | :----: | :-------------------: | :----: | :----------------------------------- |
| spring.thymeleaf.enabled              |  bool  |         true          |  默认  | 是否启用                             |
| spring.thymeleaf.mode                 | String |         HTML          |  默认  | 模板类型，可以设置为HTML5            |
| spring.thymeleaf.cache                |  bool  |         true          |  默认  | 是否启用缓存，生成环境建议设置为true |
| spring.thymeleaf.prefix               | String | classpath:/templates/ |  默认  | 模版存放路径                         |
| spring.thymeleaf.suffix               | String |         .html         |  默认  | 模版后缀                             |
| spring.thymeleaf.servlet.content-type | String |       text/html       |  默认  | Content-Type 值                      |
| spring.thymeleaf.encoding             | String |           -           | utf-8  | 模版编码                             |

## 基础使用

在`<html>`中引入thymeleaf命名空间`xmlns:th="http://www.thymeleaf.org"`，即

`<html lang="en" xmlns:th="http://www.thymeleaf.org">`

### 标签使用

#### th:text基础信息输出/th:utext html内容输出

使用"th:text"是对内容的原样输出，使用“th:utext”可以进行html标签输出。

java代码：

```java
@GetMapping("/")
public ModelAndView index(){
    ModelAndView mv=new ModelAndView("/index");
    mv.addObject("name","Hello World!");
    mv.addObject("data","<span style='color:red'>世界你好！</span>");
    return mv;
}
```

html代码：

```xml
<h2 th:text="'th:text '+${data}"></h2>
<h2 th:utext="'th:text '+${data}"></h2>
<span th:text="${name}"></span>
```

输出：

```
th:text <span style='color:red'>世界你好！</span>
th:text 世界你好！
Hello World!
```

#### th:if,th:unless条件判断

th:if为满足条件的业务处理，th:unless正好相反，是除去的意思。

java代码：

```java
@GetMapping("/")
public ModelAndView index(){
    ModelAndView mv=new ModelAndView("/index");
    mv.addObject("age",18);
    mv.addObject("age1",20);
    return mv;
}
```

html代码：

```xml
<span th:if="${age1 gt 18}" th:text="成年">成年人</span>  <br/>
<span th:unless="${age gt 18}" th:text="未成年">未成年人</span>
```

输出：

```
成年
未成年
```

#### th:switch, th:case 多条件判断

默认选项使用`th:case="*"` 指定。

html代码：

```html
<div th:switch="${age}">
    <span th:case="18" th:text="18岁"></span>
    <span th:case="19" th:text="19岁"></span>
    <span th:case="*" th:text="其他"></span>
</div>
```

#### th:each 循环

java代码：

```java
@GetMapping("/")
public ModelAndView index(){
    ModelAndView mv=new ModelAndView("/index");
    List<String> names=new ArrayList<>();
    names.add("java");
    names.add("spring");
    names.add("nodejs");
    names.add("Hello");
    mv.addObject("names",names);
    return mv;
}
```

html代码：

```xml
<div th:each="name,item:${names}">
    <span th:text="${item.count}"></span>
    <span th:text="${name}"></span>
</div>
```

输出：

```
1 java
2 spring
3 nodejs
4 Hello 
```

其中item为每行的详细值，key值如下：

- index 下标，从0开始
- count 第x个，从1开始
- size 这个集合的大小
- current 当前行的值
- even/odd:布尔值，当前循环是否是偶数/奇数（从0开始计算）
- first:布尔值，当前循环是否是第一个
- last:布尔值，当前循环是否是最后一个 

#### th:fragment、th:insert、th:replace、th:include 代码片段复用

- th:fragment标签是声明代码片段，用于解决代码复用的问题，好比Java程序写的公用代码一样，每个需要的地方都可以直接调用；
- th:insert 引用fragment的代码，保留自己的主标签；
- th:replace 引用fragment的代码，不保留自己的主标签；
- th:include 使用类似th:replace，Thymeleaf3.0之后不推荐使用；

`::`用来完成对页面片段的引用，页面片段引用时支持传参；

html代码：

```xml
<!-- footer.html -->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<div th:fragment="copyright(author)">
    <span th:text="'@CopyRight '+${author}"></span>
</div>
<div th:fragment="about(s1,s2)">
    关于：<span th:text="${s1}"></span><br/>
    <span th:text="${s2}"></span>
</div>
<div th:fragment="links(s1,s2)">
    友情链接：
    <div th:if="${s2 lt 18}">
        <span th:text="${s2}+'小于18：'+${s1}"></span>
    </div>
    <div th:unless="${s2 lt 18}">
        <span th:text="${s2}+'大于等于18：'+${s1}"></span>
    </div>
</div>
</body>
</html>
    
<!-- index.html -->
<div th:replace="footer :: copyright('Hello')"></div>
<div th:insert="footer :: about('时间简史','岁月无门')"></div>
<div th:include="footer :: links('世界帮',18)"></div>
```

输出：

```
<div>
    <span>@CopyRight Hello</span>
</div>
<div><div>
    关于：<span>时间简史</span><br/>
    <span>岁月无门</span>
</div></div>
<div>
    友情链接：
    
    <div>
        <span>18大于等于18：世界帮</span>
    </div>
</div>
```

#### th:with 定义局部变量

html代码：

```html
<div th:with="sum=1+2+3+4">
    <span th:text="${sum}"></span>
</div>
```

输出：

```
10
```

#### th:remove 删除标签

th:remove用于html代码的删除，th:remove值有五个：

- all 删除本段所有代码
- body 删除主标签内的所有元素
- tag 删除主标签，保留主标签所有的元素
- all-but-first 保留主标签和第一个元素，其他全部删除
- none 不删除任何标签

html代码：

```html
<div id="all" th:remove="all">
    <span>all</span>
    <span>1</span>
</div>

<div id="body" th:remove="body">
    <span>body</span>
    <span>2</span>
</div>

<div id="tag" th:remove="tag">
    <span>tag</span>
    <span>3</span>
</div>


<div id="all-but-first" th:remove="all-but-first">
    <span>all-but-first</span>
    <span>4</span>
</div>

<div id="none" th:remove="none">
    <span>none</span>
    <span>5</span>
</div>
```

输出：

```
<div id="body"></div>


<span>tag</span>
<span>3</span>



<div id="all-but-first">
<span>all-but-first</span>

</div>

<div id="none">
<span>none</span>
<span>5</span>
</div>
```

#### 其他标签

- th:style 定义样式 `<div th:style="'color:'+${skinColor}">`
- th:onclick 点击事件 `<input type="button" value=" Click " th:onclick="'onsub()'">`
- th:href 赋值属性href `<a th:href="${myhref}"></a>`
- th:value 赋值属性value`<input th:value="${user.name}" />`
- th:src 赋值src `<img th:src="${img}" />`
- th:action 赋值属性action `<form th:action="@{/suburl}">`
- th:id 赋值属性id `<form id="${fromid}">`
- th:attr 定义多个属性 `<img th:attr="src=@{/img/stone.jpg},alt=${alt}" />`
- th:object 定义一个对象` <div th:object="${user}">`
- ...

### 表达式使用

#### 简单表达式

- 变量表达式：${...}
- 选择变量表达式：*{...}
- 消息表达式：#{...}
- 链接表达式：@{...}
- 片段表达式：~{...}

#### 数据类型

- 文字：'one text', 'Another one!',…
- 数字文字：0, 34, 3.0, 12.3,…
- 布尔文字：true, false
- NULL文字：null
- 文字标记：one, sometext, main,…

#### 文本操作

- 字符串拼接：+
- 字面替换：|The name is ${name}|

#### 算术运算

- 二进制运算符：+, -, *, /, %
- 减号(一元运算符)：-

#### 布尔运算

- 二进制运算符：and, or
- 布尔否定(一元运算符)：!, false

#### 条件运算符

- 比较值：> gt, < lt, >= ge, <= le
- 相等判断： ==, !=

#### 条件判断

- 如果-然后：(if) ? (then)
- 如果-然后-否则：(if) ? (then) : (else)
- 违约：(value) ?: (defaultvalue)

### 使用示例

#### 变量表达式${...}

前面多次使用

#### 选择表达式

java代码：

```java
// User类
public class User {
    private Long id;
    private String name;
    private String time;
    private List<String> hobby;
    private Date date=new Date();
}

// 控制器
@GetMapping("/")
public ModelAndView index(){
    ModelAndView mv=new ModelAndView("/index");
    List<String> names=new ArrayList<>();

    Date date=new Date();
    SimpleDateFormat simpleDateFormat=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String s=simpleDateFormat.format(date);

    names.add("java");
    names.add("spring");
    names.add("nodejs");
    names.add("Hello");

    User user=new User();
    user.setId(10L);
    user.setName("ZS");
    user.setTime(s);
    user.setHobby(names);

    mv.addObject("user",user);
    return mv;
}
```

html代码：

```html
<div th:object="${user}">
    <span th:text="${#dates.format(user.getDate(),'yyyy-MM-dd HH:mm:ss')}"></span>
    <span th:text="*{id}"></span>
    <span th:text="*{name}"></span>
    <span th:text="*{time}"></span>
    <ul th:each="h,item:*{hobby}">  <!-- item前面的单个元素变量h不能少 -->
        <li th:text="${item.count}+${item.current}"></li>
    </ul>
</div>
```

输出：

```
    <div>
        <span>2020-04-01 15:11:57</span>
        <span>10</span>
        <span>ZS</span>
        <span>2020-04-01 15:11:57</span>
        <ul>
            <li>1java</li>
        </ul>
        <ul>
            <li>2spring</li>
        </ul>
        <ul>
            <li>3nodejs</li>
        </ul>
        <ul>
            <li>4Hello</li>
        </ul>
    </div>
```

#### 链接表达式

html代码：

```html
<a th:href="@{/footer(id=666,name=hello)}">链接</a>

<!-- 输出 -->
<a href="/footer?id=666&name=hello">链接</a>
```

链接表达式，可以传递参数，用逗号分隔。

#### 文本操作

文本拼接：`<span th:text="'id:'+${id}+' and name:'+${name}">ggg</span>`

文本替换：`<span th:text="|id:${id} and name:${name}|">ggg</span>`

#### 嵌入文本标签

嵌入文本有两种写法“[[...]]”和“[(...)]”，分别的作用就像th:text 和 th:utext 一样，例如：

### 表达式对象

##### 2.3.1 表达式基本对象

- `#ctx`: 操作当前上下文.
- `#vars:` 操作上下文变量.
- `#request`: (仅适用于Web项目) `HttpServletRequest`对象.
- `#response`: (仅适用于Web项目)  `HttpServletResponse` 对象.
- `#session`: (仅适用于Web项目)  `HttpSession` 对象.
- `#servletContext`: (仅适用于Web项目)  `ServletContext` 对象.

##### 2.3.2 表达式实用工具类

- `#execInfo`: 操作模板的工具类，包含了一些模板信息，比如：`${#execInfo.templateName}` .
- `#uris`: url处理的工具
- `#conversions`: methods for executing the configured *conversion service* (if any).
- `#dates`: 方法来源于 `java.util.Date` 对象，用于处理时间，比如：格式化.
- `#calendars`: 类似于 `#dates`, 但是来自于 `java.util.Calendar` 对象.
- `#numbers`: 用于格式化数字.
- `#strings`: methods for `String` objects: contains, startsWith, prepending/appending, etc.
- `#objects`: 普通的object对象方法.
- `#bools`: 判断bool类型的工具.
- `#arrays`: 数组操作工具.
- `#lists`: 列表操作数据.
- `#sets`: Set操作工具.
- `#maps`: Map操作工具.
- `#aggregates`: 操作数组或集合的工具.