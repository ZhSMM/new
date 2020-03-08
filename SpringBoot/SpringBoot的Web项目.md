# Spring Boot的Web项目

## Web

### web依赖

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

### 示例

```java
// 创建Controller
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

// 相当于@Controller与@ResponseBody两个注解
@RestController
public class HelloController {
    // 相当于@RequestMapping(method = RequestMethod.GET )
    @GetMapping("/hello")
    public String hello(){
        return "Hello ,This is your first SpringBoot Web Project!";
    }
}

// 运行，访问http://localhost:8080/hello
```

## WebFlux

> WebFlux是一个非阻塞的Web框架，不再完全依赖于Servlet，而是实现了Reactive Streams规范。也就是说，可以使用响应式编程，默认推荐的是Netty这种异步容器。

### WebFlux依赖

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
```

### 创建一个处理方法类

> 新建类HelloHandle，创建一个hello方法供接下来使用，其中返回值Mono<ServerResponse>作为响应对象，其中ServerResponse包含响应状态、响应头信息等，类上面的@Component注解用于将类实例化到Spring容器中。总的来说，这个方法就是返回一句字符串；

```java
@Component
public class HelloHandle{
    public Mono<ServerResponse> hello(ServerRequest request){
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON)
            .body(BodyInserters.fromObject("Hello,This is a Spring Boot!"));
    }
}
```

> 创建一个Router类
>
> 创建一个HelloRouter类，用来定义路由信息，每个路由都会映射到对应的处理方法（功能类似于@RequestMapping）。当接受到对应HTTP请求后，调用此方法，通过RouterFunctions.route(RequestPredicate，HandlerFunction)提供一个路由器函数的默认实现；

```java
@Configuration
public class HelloRouter{
    @Bean
    public RouterFunction<ServerResponse> routeHello(HelloHandle helloHandle){
        return RouterFunctions
            .route(RequestPredicates.GET("/hello")
            .and(RequestPredicates.accept(MediaType.APPLICATION_JSON)),
                  helloHandle::hello);
    }
}

// 运行，访问http://localhost:8080/hello
```

## 使用热部署

> 引入热部署依赖，重新编译修改的类文件或配置文件等，Spring Boot框架会自动重启。

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
        </dependency>
```

## 配置文件

1. 配置文件目录：src/main/java/resources

2. 配置文件两种格式：

   - application.properties
   - application.yml

3. 自定义属性：

   ```java
   // properties文件
   book.name=Spring Boot
   book.author=Boy
   
   // 读取配置文件内容
   // 在属性上使用@Value("${属性名}")
   @RestController
   public class TestController{
       @Value("${book.name}")
       private String bookName;
       
       @Value("${book.name}")
       private String bookName;
       
       @GetMapping("test1")
       public String test1{
           return "bookName:"+bookName+",bookAuthor:"+bookAuthor;
       }
   }
   
   // 注意：在application。properties中配置中文值，读取时会出现中文乱码的问题，
   // 因为Java默认ISO-8859-1的编码方式来读取*.properties配置文件，而SpringBoot
   // 应用以UTF-8方式读取。
   // 解决办法：以Unicode方式来展示字符，如book.author=杨 配置成
   // book.author=\u6768
   ```

4. 使用随机数

   ```java
   // application.properties支持随机数random
   // 随机字符串：random.value
   book.value=${random.value}
   // 随机int值
   book.intValue=${random.int}
   //类似有：random.long、random.uuid、random.int{1000}等
   
   // 同时支持属性间引用
   book.title=书名：${book.value}
   
   // 使用JavaBean模式赋值
   // 在实体类上加上注解@ConfigrationProperties(prefix="book")
   // 在启动类上添加注解
   // @EnableConfigurationProperties(BookConfiguration.class)
   // 表明启动这个配置类
   @ConfigrationProperties(prefix="book")
   public class BookConfiguration{
       private String title;
       ……
       setter/getter
   }
   
   // @Autowired注解注入类并测试
   @RestController
   public class Test{
       @Autowired
       private BookConfiguration book;
       @GetMapping("Test")
       public BookConfiguration test(){
           return book;
       }
   }
   
   // 运行，浏览器输入localhost:8080/test
   ```

5. 多环境配置

   ```java
   // 一套程序在不同环境中发布，需要不同的配置
   application-dev.properties:开发环境
   application-test.properties:测试环境
   application-prod.properties:生产环境
       
   // 通过在主配置文件application.properties中配置spring.profiles.active来设置
   // 当前使用的配置文件(比如在主配置文件中配置本次指定使用的配置文件后缀)；
   spring.profiles.active=test
   ```

6. 自定义配置文件

   ```java
   
   // 自定义配置文件 test.properties
   com.book.name=Spring Boot
       
   // JavaBean读取配置文件
   @Component
   // 指定属性路径
   @PropertySource(value="classpath:test.properties")
   @ConfigurationProperties(prefix="com.book")
   public class Configuration{
       private String name;
       ……
   }
   ```

    

