# Spring热部署

Spring Boot配置：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>

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

idea设置：

1. 设置自动build：`File->Settings->Build,Execution,Deployment->compiler`，勾选`Build project automatically`;
2. Register配置：快捷键`Ctrl+Shift+Alt+/`,然后选择Register，再找到并勾选compiler.automake.allow.when.app.running，重启idea就能实现热部署。 

