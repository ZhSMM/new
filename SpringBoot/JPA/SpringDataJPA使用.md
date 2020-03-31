# Spring Data JPA使用

JPA（Java Persistence API）：Java持久层API，通过对象/关联映射工具来管理java中的应用关系数据。它的出现是为了简化现有的持久化开发工作和整合ORM技术， 结束各个ORM框架各自为营的局面。

> JPA仅仅是一套规范，不是一套产品，也就是说Hibernate，TopLink等是实现了JPA规范的一套产品。

Spring Data JPA：是Spring基于ORM框架、JPA规范的基础上封装的一套JPA应用框架，是基于Hibernate之上构建的JPA使用解决方案，用极简的代码实现了对数据库的访问和操作，包括了增、删、改、查等在内的常用功能。

[Spring Data JPA官方文档](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference)

## maven依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.2.RELEASE</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
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

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

## application.yaml

```yaml
spring:
  # MySQL数据源
  datasource:
    url: jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 1234

  jpa:
    properties:
      hibernate:
        hbm2ddl:
          auto: create
        dialect: org.hibernate.dialect.MySQL5InnoDBDialect
        format_sql: true
    show-sql: true
```

## application.properties

打印SQL到控制台：

spring.jpa.properties.hibernate.show_sql=true          //控制台是否打印
spring.jpa.properties.hibernate.format_sql=true        //格式化sql语句
spring.jpa.properties.hibernate.use_sql_comments=true  //指出是什么操作生成了该语句

sl4j日志（在xml配置文件中使用）：

```
<logger name="org.hibernate.SQL" level="DEBUG"/>  //该语句控制打印SQL，如果你在yaml或者properties文件里配置了“spring.jpa.properties.hibernate.show_sql=true ” ，则不需要再配置该语句

<logger name="org.hibernate.engine.QueryParameters" level="DEBUG"/>
<logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG"/>
<logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE"/>
```

```properties
# MySQL数据源
spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.username=root
spring.datasource.password=1234

# JPA配置
# 配置数据源
spring.jpa.database=mysql
# 显示SQL语句
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.hibernate.ddl-auto=create
```

spring.jpa.hibernate.ddl-auto 参数用来配置是否开启自动更新数据库表结构，可选的参数如下：

- create：每次加载hibernate时，先删除已存在的数据库表结构再重新生成；
- create-drop：每次加载hibernate时，先删除已存在的数据库表结构再重新生成，并且当 sessionFactory关闭时自动删除生成的数据库表结构；
- update：只在第一次加载hibernate时自动生成数据库表结构，以后再次加载hibernate时根据model类自动更新表结构；
- validate：每次加载hibernate时，验证数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值；
- none：关闭自动更新 。

spring.jpa.properties.hibernate.dialect：指定生成表明的存储引擎为InnoDB；

## 创建POJO

```java
import javax.persistence.*;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false,unique = true)
    private String userName;

    @Column(nullable = false)
    private String password;

    @Column(nullable = false,unique = true)
    private String email;

    @Column(nullable = true,unique = true)
    private String nickName;

    @Column(nullable = false)
    private String regTime;

    public User(String userName, String password, String email, String nickName, String regTime) {
        this.userName = userName;
        this.password = password;
        this.email = email;
        this.nickName = nickName;
        this.regTime = regTime;
    }

    public User() {
    }
    ------------setter/getter--------------
}
```

注解说明：

- @Entity(name="EntityName") 必须，用来标注一个数据库对应的实体，数据库中创建的表名默认和类名一致。其中，name 为可选，对应数据库中一个表，使用此注解标记 Pojo 是一个 JPA 实体。
- @Table(name=""，catalog=""，schema="") 可选，用来标注一个数据库对应的实体，数据库中创建的表名默认和类名一致。通常和 @Entity 配合使用，只能标注在实体的 class 定义处，表示实体对应的数据库表的信息。
- @Id 必须，@Id 定义了映射到数据库表的主键的属性，一个实体只能有一个属性被映射为主键。
- @GeneratedValue(strategy=GenerationType，generator="") 可选：strategy:  表示主键生成策略，有 AUTO、INDENTITY、SEQUENCE 和 TABLE 4 种，分别表示让 ORM  框架自动选择，generator: 表示主键生成器的名称。
- @Column(name = "user_code"， nullable = false， length=32) 可选，@Column  描述了数据库表中该字段的详细定义，这对于根据 JPA 注解生成数据库表结构的工具。
  - name:  表示数据库表中该字段的名称，默认情形属性名称一致；
  - nullable: 表示该字段是否允许为 null，默认为 true；
  - unique:  表示该字段是否是唯一标识，默认为 false；
  - length: 表示该字段的大小，仅对 String 类型的字段有效。
- @Transient可选，@Transient 表示该属性并非一个到数据库表的字段的映射，ORM 框架将忽略该属性。
- @Enumerated 可选，使用枚举的时候，我们希望数据库中存储的是枚举对应的 String 类型，而不是枚举的索引值，需要在属性上面添加 @Enumerated(EnumType.STRING) 注解。

## Repository

继承JpaRepository类，自动继承了以下方法：

- Repository.save(user); // 插入或保存
- Repository.saveFlush(user); // 保存并刷新
- Repository.exists(1) // 主键查询是否存在
- Repository.findOne(1); // 主键查询单条
- Repository.delete(1); // 主键删除
- Repository.findByUsername("stone"); // 查询单条
- Repository.findAll(pageable); // 带排序和分页的查询列表
- Repository.saveState(1, 0); // 更新单个字段

```java
import org.example.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User,Long> {
    User findByUserNameOrEmail(String userName,String email);
    User findByUserName(String userName);
}
```

## 测试

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// 配置类
@SpringBootApplication
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class,args);
    }
}
```

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test {
    @Resource
    private UserRepository userRepository;

    @org.junit.Test
    public void test(){
        Date data = new Date();
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String formattedDate = dateFormat.format(data);

        userRepository.save(new User("aa","aa123456","aa@126.com","aa",formattedDate));
        userRepository.save(new User("bb","bb123456","bb@126.com","bb",formattedDate));
        userRepository.save(new User("cc","cc123456","cc@126.com","cc",formattedDate));

        Assert.assertEquals(3,userRepository.findAll().size());
        Assert.assertEquals("bb",userRepository.findByUserNameOrEmail("bb","bb@126.com").getNickName());
        userRepository.delete(userRepository.findByUserName("aa"));
    }
}
```

## 查询语句

### 自定义查询

Spring Data JPA 可以根据接口方法名来实现数据库操作，主要的语法是  findXXBy、readAXXBy、queryXXBy、countXXBy、getXXBy 后面跟属性名称，利用这个功能仅需要在定义的  Repository 中添加对应的方法名即可，使用时 Spring Boot 会自动帮我们实现。

```java
// 根据用户名查询用户：
User findByUserName(String userName);
// 也可以加一些关键字 And、or：
User findByUserNameOrEmail(String username,String email);
// 修改、删除、统计也是类似语法：
Long deleteById(Long id);
Long countByUserName(String userName);
// 基本上 SQL 体系中的关键词都可以使用，如 LIKE 、IgnoreCase、OrderBy：
List<User> findByEmailLike(String email);
User findByUserNameIgnoreCase(String userName);
List<User> findByUserNameOrderByEmailDesc(String email);
```

关键字的使用和生产SQL：

|       关键字        |                             示例                             |                          JPQL 片段                           |
| :-----------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|        `And`        |                 `findByLastnameAndFirstname`                 |        `… where x.lastname = ?1 and x.firstname = ?2`        |
|        `Or`         |                 `findByLastnameOrFirstname`                  |        `… where x.lastname = ?1 or x.firstname = ?2`         |
|     `Is,Equals`     | `findByFirstname`,`findByFirstnameIs`,`findByFirstnameEquals` |                  `… where x.firstname = ?1`                  |
|      `Between`      |                   `findByStartDateBetween`                   |           `… where x.startDate between ?1 and ?2`            |
|     `LessThan`      |                     `findByAgeLessThan`                      |                     `… where x.age < ?1`                     |
|   `LessThanEqual`   |                   `findByAgeLessThanEqual`                   |                    `… where x.age <= ?1`                     |
|    `GreaterThan`    |                    `findByAgeGreaterThan`                    |                     `… where x.age > ?1`                     |
| `GreaterThanEqual`  |                 `findByAgeGreaterThanEqual`                  |                    `… where x.age >= ?1`                     |
|       `After`       |                    `findByStartDateAfter`                    |                  `… where x.startDate > ?1`                  |
|      `Before`       |                   `findByStartDateBefore`                    |                  `… where x.startDate < ?1`                  |
|      `IsNull`       |                      `findByAgeIsNull`                       |                   `… where x.age is null`                    |
| `IsNotNull,NotNull` |                    `findByAge(Is)NotNull`                    |                   `… where x.age not null`                   |
|       `Like`        |                    `findByFirstnameLike`                     |                `… where x.firstname like ?1`                 |
|      `NotLike`      |                   `findByFirstnameNotLike`                   |              `… where x.firstname not like ?1`               |
|   `StartingWith`    |                `findByFirstnameStartingWith`                 | `… where x.firstname like ?1` (parameter bound with appended `%`) |
|    `EndingWith`     |                 `findByFirstnameEndingWith`                  | `… where x.firstname like ?1` (parameter bound with prepended `%`) |
|    `Containing`     |                 `findByFirstnameContaining`                  | `… where x.firstname like ?1` (parameter bound wrapped in `%`) |
|      `OrderBy`      |                `findByAgeOrderByLastnameDesc`                |        `… where x.age = ?1 order by x.lastname desc`         |
|        `Not`        |                     `findByLastnameNot`                      |                  `… where x.lastname <> ?1`                  |
|        `In`         |                `findByAgeIn(Collection ages)`                |                    `… where x.age in ?1`                     |
|       `NotIn`       |               `findByAgeNotIn(Collection age)`               |                  `… where x.age not in ?1`                   |
|       `True`        |                     `findByActiveTrue()`                     |                  `… where x.active = true`                   |
|       `False`       |                    `findByActiveFalse()`                     |                  `… where x.active = false`                  |
|    `IgnoreCase`     |                 `findByFirstnameIgnoreCase`                  |           `… where UPPER(x.firstame) = UPPER(?1)`            |

### 自定义Sql语句查询

Spring Boot JPA可以通过@Query(sql)来自定义sql语句：

示例：

```java
// 注意：在执行修改和删除的时候必须添加@Modifying注解，ORM才知道要执行写操作，
//  update/delete query 的时候，也必须需要加上@Transactional（事务）才能正常操作。
@Transactional
@Modifying
@Query("update User set name=?1 where id=?2")
public int modifyName(String name,Long id);
```

### 使用事务

步骤一：配置application.properties配置数据库引擎为InnoDB：

```
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
```

查看MySQL数据库的引擎：`show table status from table_name;`

修改表的引擎：`alter table table_name engine=innodb;`

步骤二：在方法或类上标识事务@Transactional

示例：

```java
import javax.transaction.Transactional;
@Transactional
public void saveGroup(User user1,User user2){
    userRepository.save(user1);
    userRepository.save(user2);
}
```

### 常见错误

1. No default constructor for entity：实体类Entity没有空参数的默认构造函数，新增即可解决。

2. java.sql.SQLException: Access denied for user ''@'172.17.0.1' (using password: NO)：启动项目报错，用户名和密码配置的key有误，MySQL8的用户名和密码配置和之前的不一样，MySQL 8 正确的用户名密码配置如下：

   ```
   spring.datasource.username=root
   spring.datasource.password=123456
   # 以下为配置老数据库驱动配置
   #spring.datasource.data-username=root
   #spring.datasource.data-password=123456
   ```

3. Caused by: java.lang.IllegalStateException: Cannot load driver class: com.mysql.jdbc.Driver：MySQL 8 的spring.datasource.driver-class-name配置需要改为“com.mysql.cj.jdbc.Driver”而不是“com.mysql.jdbc.Driver”，正确配置如下：

   ```
   spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
   ```

## 参考

1. [https://www.cnblogs.com/wadmwz/p/10313495.html](https://www.cnblogs.com/wadmwz/p/10313495.html)