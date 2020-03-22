# 通过java代码装配Bean

尽管在很多场景下通过组件扫描和自动装配实现Spring的自动化配置是更为推荐的 方式，但有时候自动化配置的方案行不通，因此需要明确配置Spring。比如说，你想要将第三方库中的组件装配到你的应用中，在这种情况下，是没有办法在它的类上添加@Component和@Autowired注解的，因此就不能使用自动化装配的方案了。

显示装配有两种方案：Java注解和xml配置。

- Java注解更为强大，类型安全且对重构友好，推荐使用。
- xml注解不支持类型检查。

## 创建配置类

```java
import org.springframework.context.annotation.Configuration;

@Configuration
public class NoAutoConfig {
}
```

@Configuration注解表明这个类是一个配置类，该类应该包含在Spring应用上下文中如何创建bean的细节。

声明简单的Bean：

```java
private Random random=new Random();
@Bean
public ICar setCar(){
    return random.nextInt(50)>25? new Car():new Bicycle();
}
```

- @Bean注解会告诉Spring这个方法将会返回一个对象，该对象要注册为Spring应用上 下文中的bean。方法体中包含了最终产生bean实例的逻辑;
- 默认情况下，bean的ID与带有@Bean注解的方法名是一样的。在本例中，bean的名字setCar。如果你想为其设置成一个不同的名字的话，那么可以重命名该方法，也可以通过name属性指定一个不同的名字;

## 注入bean

1. 构造方法注入：

   ```java
   @Bean
   public IDriver driver(){
       return new Driver(setCar());
   }
   ```

2. Setter方法注入：

   ```java
   @Bean
   public IDriver driver(ICar car){
       return new Driver(car);
   }
   ```

   

