# 04Java项目build

```
// 使用插件
plugins {
   id 'org.springframework.boot' version '2.2.5.RELEASE'
   id 'io.spring.dependency-management' version '1.0.9.RELEASE'
   id 'java'
}

group = 'top.songfang'

// 指定了生成的打包的文件名称
version = '0.0.1-SNAPSHOT'

// 指定编译 .java 文件的JDK版本
sourceCompatibility = '1.8'

repositories {
   // 配置阿里云镜像下载
   maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
   maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
   // 默认中央仓库
   mavenCentral()
}

// 依赖关系
dependencies {
   implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
   implementation 'org.springframework.boot:spring-boot-starter-web'
   implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
   implementation 'org.springframework.boot:spring-boot-starter-data-elasticsearch'
   compile 'net.java.dev.jna:jna:5.5.0'
   runtimeOnly 'mysql:mysql-connector-java'
   testImplementation('org.springframework.boot:spring-boot-starter-test') {
      exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
   }
}
```