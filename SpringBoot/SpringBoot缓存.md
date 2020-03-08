# Spring Boot缓存

> 数据库不能完全高性能的解决任何事情，因此出现了缓存。缓存数据交换的缓冲区，一般来说会将访问量比较大的数据从数据库中查询出来放入缓存中，当下次获取数据时，直接从缓存中获取。通常缓存会放入内存或硬盘中，方便开发者使用。

## Spring Cache简介

Spring Cache是Spring 3.1后引入的新技术。它并不像正常缓存那样存储数据，其核心思想是：当我们在调用一个缓存方法时，会把该方法参数和返回结果作为一个键值对存放在缓存中，等到下次利用同样的参数来调用该方法时将不再执行该方法，而是直接从缓存中获取结果进行返回，从而实现缓存的功能。

Spring Cache的使用：与Spring对于事物管理的使用类似，可以基于注解使用或者基于XML配置方式使用。

### 基于注解

1. @Cacheable

   @Cacheable注解用于标记缓存，也就是说对使用@Cacheable注解的位置进行缓存。该注解可以在方法和类上进行标记。

   - 对方法进行标记时，表明此方法支持缓存；
   - 对类进行标记时，表明当前类中所有的方法都支持缓存。

   在支持Spring Cache的环境下，对于使用@Cacheable标记的方法，Spring在每次调用方法前都会根据key查询当前Cache中是否存在相同key的缓存元素，如果存在，就不再执行该方法，而是直接从缓存中获取结果进行返回，否则执行该方法并将结果存入指定的缓存中。在使用Cacheable时经常搭配以下3个属性进行使用：

   - value：在使用@Cacheable注解的时候value属性必须指定，用于指定Cache的名称，即表明当前缓存的返回值用于哪个缓存上；

   - key：和名称一样，用于指定缓存对应的key。key属性不是必须指定的，当没有指定key，Spring就会为我们使用默认策略生成的key。

     - 默认策略：当前缓存方法没有参数，那么当前key为0；如果当前缓存方法有一个参数，那么以key为参数值；如果当前缓存方法有多个参数，那么key为所有参数的hashcode值。

     - 用Spring提供的EL表达式来指定当前缓存方法的key。一般而言，我们可以使用当前缓存方法的参数指定key，一般为"#参数名"。如果参数为对象，就可以使用对象的属性指定key，例：

       ```java
       @Cacheable(value="users",key="#user.id")
       public User findUser(User user){
           return new User();
       }
       ```

     - Spring框架提供了root对象来使用key属性，在指定key属性时可以忽略#root，因为Spring默认调用的就是root对象的属性。root对象内置了如下几个属性：

       1. methodName:当前方法的名称，key值为#root.methodName或methodName；
       2. method：指定当前方法，key值为#root.method.name或method.name;
       3. target：当前被调用的对象，key值为#root.target或target；
       4. targetClass：当前被调用对象的class，key值为#root.targetClass或targetClass;
       5. args：当前方法参数组成的数组，key值为#root.args[0]或args[0];
       6. caches：当前被调用的方法使用的Cache，key值为#root.caches[0].name或caches[0].name;

   - condition：主要用于指定当前缓存的触发条件。

2. @CachePut

   该注解用于将标记该注解的方法的返回值放入缓存中，无论缓存中是否包含当前缓存，只是以键值的形式将执行结果放入缓存中。在使用方面与@Cacheable一致。

3. @CacheEvict

   用于清除缓存数据，与@Cacheable类似；@CacheEvict用于方法时清除当前方法的缓存，用于类时清除当前类所有方法的缓存。@CacheEvict除了提供与@Cacheable一致的3个属性外，还提供了一个常用的属性allEntries，该属性默认值为false，如果指定属性值为true，就会清除当前value值的所有缓存。

## 配置依赖

```java
    <dependencies>
        <!-- 缓存依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <!-- JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- MySQL -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

