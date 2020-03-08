# 03创建Gradle Builds

## 前置条件

- 一个终端应用程序（比如cmd）
- 版本1.8以上的JRE或JDK
- Gradle distribution，4.10.3版本往上

## 初始化项目

### 创建项目路径

```
> mkdir basic-demo
> cd basic-demo
```

### 生成简单项目

使用`gradle init`命令来生成一个简单项目，可以选择两种模板：Groovy或Kotlin，可以在生成时指定：`gradle init --dsl kotlin`

```
❯ gradle init 
Starting a Gradle Daemon (subsequent builds will be faster)

Select type of project to generate:
  1: basic
  2: application
  3: library
  4: Gradle plugin
Enter selection (default: basic) [1..4] 1

BUILD SUCCESSFUL in 3s
2 actionable tasks: 2 executed
```

#### 可选参数

- `--quiet`：执行过程中，只显示Error级别的输出；
- `--stacktrace`：执行过程中，输出所有Exception的stacktrace；
- `--full-stacktrace`，执行过程中，输出所有Exception的完整stacktrace；
- `--offline`，离线模式，不使用网络资源；
- `--no-daemon`，不使用Deamon。

### 项目目录结构

Groovy模板生成的结构：

```
├── build.gradle  // 用于配置当前项目的Gradle构建脚本
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar   // Gradle Wrapper的jar包
│       └── gradle-wrapper.properties  // Gradle Wrapper配置属性
├── gradlew  // 用于基于unix的系统的Gradle Wrapper脚本
├── gradlew.bat  // 用于基于Windows的系统的Gradle Wrapper脚本
└── settings.gradle  // 用于配置Gradle build的Gradle设置脚本
```

Kotlin模板生成的结构：

```
├── build.gradle.kts  // 用于配置当前项目的Gradle构建脚本
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar  
│       └── gradle-wrapper.properties  
├── gradlew  
├── gradlew.bat  
└── settings.gradle.kts  
```