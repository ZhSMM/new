# mybatis

MyBatis官方GitHub网址：[https://github.com/mybatis/](https://github.com/mybatis/)

- mybatis-3：MyBatis源码
- generator：代码生成器，可以生成一些常见的基本方法，提高工作效率
- ehcache-cache：默认集成Ehcache的缓存实现
- redis-cache：默认集成Redis的缓存实现
- spring：方便和Spring集成的工具类
- mybatis-spring-boot：方便和Spring Boot集成的工具类

官方文档：[https://mybatis.org/mybatis-3/zh/index.html](https://mybatis.org/mybatis-3/zh/index.html)

## 概述

### maven依赖

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
</dependency>
```

### 从 XML 中构建 SqlSessionFactory