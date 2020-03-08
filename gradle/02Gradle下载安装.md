# Gradle下载安装

您可以在Linux、macOS或Windows上安装Gradle构建工具。本文档介绍了如何使用像SDKMAN!这样的包管理器进行安装或自制，以及手动安装。

官方下载网址：[https://gradle.org/releases/](https://gradle.org/releases/)

官方推荐Gradle升级方式：Gradle Wrapper

### 前置条件

- Gradle运行在所有主要的操作系统上，并且只需要一个Java开发工具包版本8或更高就可以运行。要进行检查，请运行java -version，输出如下：

  ```
  > java -version
  java version "1.8.0_241"
  Java(TM) SE Runtime Environment (build 1.8.0_241-b07)
  Java HotSpot(TM) 64-Bit Server VM (build 25.241-b07, mixed mode)
  ```

- Gradle附带了自己的Groovy库，因此不需要安装Groovy。Gradle会忽略任何现有的Groovy安装。

- Gradle使用它在你的路径中找到的任何JDK。或者，您可以设置JAVA_HOME环境变量来指向所需JDK的安装目录。

### 安装

#### 使用包管理器

Linux环境：SDKMAN！ `> sdk install gradle`

macOS环境：Homebrew `>brew install gradle`

#### 手动安装

1. 下载：[https://gradle.org/releases/](https://gradle.org/releases/)

   - Binary-only (bin)
   - Complete (all) with docs and sources

2. 解压

   - Linux&macOS环境：

     ```
     > mkdir /opt/gradle  zip文件目录
     > unzip -d /opt/gradle gradle-6.2.1-bin.zip
     > ls /opt/gradle/gradle-6.2.1
     LICENSE  NOTICE  bin  README  init.d  lib  media
     ```

   - Windows：将下载后的压缩包解压指定目录，比如（D:\Gradle）

3. 设置环境变量

   - Linux&macOS环境：`export PATH=$PATH:/opt/gradle/gradle-6.2.1/bin`
   - Windows：Path变量新增`D:\Gradle\gradle-6.2.1\bin`

4. 验证：命令行：`gradle -v`

