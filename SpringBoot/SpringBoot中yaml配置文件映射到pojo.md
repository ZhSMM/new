# Spring Boot中yaml配置文件映射到pojo

Spring Boot工程下自动加载src/main/resources/application.yaml或application.yml，默认读取application.yml或application.yaml的配置到POJO对象，也可以通过实现yaml配置文件加载工厂，以使用@PropertySource注解加载指定的yaml文件的配置映射到POJO对象。

官方文档：[https://docs.spring.io/spring-boot/docs/2.1.9.RELEASE/reference/html/configuration-metadata.html#configuration-metadata-annotation-processor](https://docs.spring.io/spring-boot/docs/2.1.9.RELEASE/reference/html/configuration-metadata.html#configuration-metadata-annotation-processor)

## application.yaml

maven配置：通过@ConfigurationProperties注解，使用spring-boot-configuration-processor包可以轻松配置元数据。

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-configuration-processor</artifactId>
	<optional>true</optional>
</dependency>
```

在Java的POJO上添加@Component注解和@ConfigurationProperties(prefix = "user")注解；

## 自定义的yaml配置文件

需要通过@PropertySource属性指定自定义的路径文件，且需要自定义yaml文件的加载类。

### @PropertySource 注解

使用 @PropertySource 注解，可以编写配置文件的映射类，取代 @Value，在需要用到的地方，用 @Autowire 自动装配配置类即可。

```java
public @interface PropertySource {
    /** 加载资源的名称 */
    String name () default "";
    /** 
     *  加载资源的路径
     *    1. 可使用 classpath, 如:"classpath:/config/test.yml"
     *    2. 如有多个文件路径放在 {} 中，使用 ',' 号隔开，如：
     *      {"classpath:/config/test1.yml","classpath:/config/test2.yml"}
     *    3. 除使用 classpath 外，还可使用文件的地址，如：
     *      "file:/rest/application.properties"
     */
    String [] value ();
    // 此属性为根据资源路径找不到文件后是否报错，默认为是 false
    boolean ignoreResourceNotFound () default false;
    // 此为读取文件的编码，若配置中有中文建议使用'utf-8'
    String encoding () default "";
    // 解析配置文件的工厂类，默认PropertySourceFactory.class
    Class<? extends PropertySourceFactory> factory () default PropertySourceFactory.class;
}
```

### PropertySourceFactory 接口

默认使用 @PropertySource 可以指定解析 properties 和 xml 配置文件，却不可以解析 yaml 配置文件，因为 spring 默认只有一个解析 properties 和 xml 文件的工厂类 DefaultPropertySourceFactory，没有解析 yaml 的工厂类，但是我们可以通过实现 PropertySourceFactory 接口自己写。

PropertySourceFactory 接口源码：发现其中只有一个方法 createPropertySource ()，参考 DefaultPropertySourceFactory 类编写 yaml 解析工厂类。

```java
// PropertySourceFactory源码
public interface PropertySourceFactory {
    PropertySource<?> createPropertySource(@Nullable String var1, EncodedResource var2) throws IOException;
}
// DefaultPropertySourceFactory源码
public class DefaultPropertySourceFactory implements PropertySourceFactory {
    public DefaultPropertySourceFactory() {
    }

    public PropertySource<?> createPropertySource(@Nullable String name, EncodedResource resource) throws IOException {
        return name != null ? new ResourcePropertySource(name, resource) : new ResourcePropertySource(resource);
    }
}
```

### 编写 yaml 解析工厂类

引入Maven依赖：spring 解析 yaml 依赖此包，不引入，用 maven-jar-plugin 打 jar 包后会出错

```xml
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
</dependency>
```

自定义工厂类 YamlPropertySourceFactory：

```java
public class YamlPropertySourceFactory implements PropertySourceFactory {
    public YamlPropertySourceFactory() {
    }

    public PropertySource<?> createPropertySource(@Nullable String name, EncodedResource resource) throws IOException {
        // 返回yaml资源
        return new YamlPropertySourceLoader().load(
            resource.getResource().getFilename(), resource.getResource()).get(0);
    }
}
```

### 使用

POJO：

```java
@Component
@PropertySource(value = "classpath:yaml/user.yaml",encoding = "UTF-8")
@ConfigurationProperties(prefix = "user")
public class User {
    private String name;
    private List<String> hobby;
    private Map<String[],Integer> count;
    private Date date;
    private String[] hello;
```

user.yaml：

```yaml
user:
  name: 张三
  hobby: [下棋,游戏,钓鱼]
  date: 2020/04/02
  hello:
    - 世界
    - 老师
    - 宇宙
  count:
    ?
      - Hello
      - World
    : 16
    ?
      - good
      - very
    : 19
```

Test：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
    @Autowired
    private User user;

    @org.junit.Test
    public void test(){
        System.out.println(user);
    }
}
```

