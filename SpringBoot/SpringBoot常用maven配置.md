# Spring Boot常用maven配置

官方Starter：[https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#using-boot](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#using-boot)

pom.xml

```xml
    <!-- 继承springboot父项目,依赖会自动匹配版本号 -->
	<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/>
    </parent>

    <dependencies>
        
		<!-- 引入springboot的web支持 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- yaml中属性自动注入pojo时需要，能够进行提示 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- 测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        
        <!--引入devtools全局热部署，谨慎使用，和redis反序列化有缓存冲突，反序列化时报错-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
        
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <fork>true</fork>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

