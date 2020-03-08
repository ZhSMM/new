# SpringMVC

## 框架原理

mvc：设计模式

- M：模型，包括pojo、action、service及dao等；
- V：视图渲染
- C：控制器，接受用户请求（request）

springMVC：

1. 发起请求到前端控制器（DispatcherServlet）；
2. 前端控制器请求HandlerMapping查找Handler（根据xml配置、注解）；
3. 处理器映射器HandlerMapping向前端控制器返回Handler；
4. 前端控制器调用处理器适配器去执行Handler；
5. 处理器适配器去执行Handler；
6. Hander执行完给适配器返回ModelAndView；
7. 处理器适配器向前端控制器返回AndView（ModelAndView是springMVC一个底层对象，包括Model和View）；
8. 前端控制器请求视图解析器去进行视图解析（根据逻辑视图名解析成真正的视图）；
9. 视图解析器向前端控制器返回View；
10. 前端控制器进行视图渲染（视图渲染将数据模型（在ModelAndView中）填充到request域）。

### 前端控制器

DispatcherServlet：（减少了其他组件的耦合度）

作用：接受请求，响应结果，相当于转发器。

### 处理器映射器

HandlerMapping：

作用：根据请求的url查找Handler。

### 处理器适配器

HandlerAdapter：

作用：根据特定规则（HandlerAdapter要求的规则）去执行Handler。

### 处理器Handler（自己实现）

作用：编写Handler时要按照HandlerAdapter的要求去做，才能正确处理。

### 视图解析器

View resolver：

作用：进行视图解析，根据逻辑视图名解析成真正的视图（View）。

### 视图View

View是一个接口，实现类支持不同的View类型（jsp、freeMarker、Thymeleaf等）。
