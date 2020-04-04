# Swagger使用

swagger官网：[https://swagger.io/](https://swagger.io/)

swagger是一款API开发工具。

## 基本使用

引入maven依赖：

```xml
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
</dependencies>
```

配置swagger：

```java
// 声明配置类
@Configuration
// 启用Swagger2配置
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket docket(Environment env){
        // 获取开发环境
        Profiles profiles=Profiles.of("dev","test");
        boolean flag=env.acceptsProfiles(profiles);
        System.out.println(flag);


        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())  // 基本信息
                .enable(flag)  // 是否开启swagger
                .select()  // 选择器
                    // 选择相应项：
                    // RequestHandlerSelectors.any():选择所有
                    //  .none():一个都不选
                    //  .basePackage:根据基础包进行选择
                    //  .withClassAnnotation():选择类注解修饰的类（比如@RestController）
                    //  .withMethodAnnotation():选择方法注解修饰的方法（比如@GetMapping注解）
                    .apis(RequestHandlerSelectors.basePackage("org.example.controller"))
                    // 可以根据url路径设置哪些请求加入文档，忽略哪些请求
                    // PathSelectors:
                    //  .regex():通过url路径进行选择
                    //  .none():全都不选
                    //  .any():所有
                    //  .ant():ant表达式匹配
                    .paths(PathSelectors.regex("/hello"))
                    .build();
    }
    @Bean
    public ApiInfo apiInfo(){
        Contact contact=new Contact("张三", "https://www.songfang.top", "1911472163@qq.com");
        return new ApiInfo("Swagger Test",  // 设置文档的标题
                "第一次测试",    // 设置文档描述
                "v1.0",       // 设置文档版本
                "https://www.songfang.top",   // 文档服务地址
                contact,    // 作者信息：名字，博客网址，邮箱
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList());
    }
}
```

Controller：

Swagger 通过注解定制接口对外展示的信息，这些信息包括接口名、请求方法、参数、返回信息等。更多注解类型：

- @Api：修饰整个类，描述Controller的作用
- @ApiOperation：描述一个类的一个方法，或者说一个接口
- @ApiParam：单个参数描述
- @ApiModel：用对象来接收参数
- @ApiProperty：用对象接收参数时，描述对象的一个字段
- @ApiResponse：HTTP响应其中1个描述
- @ApiResponses：HTTP响应整体描述
- @ApiIgnore：使用该注解忽略这个API
- @ApiError ：发生错误返回的信息
- @ApiImplicitParam：描述一个请求参数，可以配置参数的中文含义，还可以给参数设置默认值
- @ApiImplicitParams：描述由多个 @ApiImplicitParam 注解的参数组成的请求参数列表

访问：[http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)

在Security中配置：当SpringBoot集成了Spring Security，在不做额外配置的话，Swagger2文档会被拦截。解决方法：

重写Security配置类的configure方法：

```java
@Override
public void configure ( WebSecurity web) throws Exception {
    web.ignoring()
      .antMatchers("/swagger-ui.html")
      .antMatchers("/v2/**")
      .antMatchers("/swagger-resources/**");
} 
```

