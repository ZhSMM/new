# spring boot起步依赖

gradle构建项目：使用方便管理依赖

```
plugins {
    id 'org.springframework.boot' version '2.2.2.RELEASE'
    id 'io.spring.dependency-management' version '1.0.8.RELEASE'
    id 'java'
}
repositories {
    maven {url "http://maven.aliyun.com/nexus/content/groups/public/" }
    mavenCentral()
}
```

覆盖起步依赖引入的依赖项：

gradle：

```
compile("org.springframework.boot:spring-boot-starter-web") {   
	exclude group: 'com.fasterxml.jackson.core' 
}
```

maven:

```xml
<dependency>   
    <groupId>org.springframework.boot</groupId>   
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>     
        <exclusion>       
            <groupId>com.fasterxml.jackson.core</groupId>     
        </exclusion>   
    </exclusions> 
</dependency> 
```

引入另外一版本的依赖项：

gradle：Gradle和Maven不太一样，Gradle 倾向于使用库的新版本。因此，如果你要使用老版本的Jackon，则不得不把老版本的依赖加入构建，并把Web起步依赖传递依赖的那个版本排除掉： 

```
compile("org.springframework.boot:spring-boot-starter-web") {
	exclude group: 'com.fasterxml.jackson.core' 
} 
compile("com.fasterxml.jackson.core:jackson-databind:2.3.1") 
```

maven：Maven总是会用近的依赖，也就是说，你在项目的构建说明文件里增加的这个依赖，会覆 盖传递依赖引入的另一个依赖。

```xml
<dependency>   
    <groupId>com.fasterxml.jackson.core</groupId>   
    <artifactId>jackson-databind</artifactId>   
    <version>2.4.3</version> 
</dependency> 
```



