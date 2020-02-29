# IntelliJ idea报错

## 问题

报错：java.lang.ClassNotFoundException: org.springframework.web.servlet.DispatcherServlet

背景：Intellij idea使用Maven搭建web项目启动报找不到类，而maven的pom.xml都已引用，问题就是在于没在WEB-INF下新建lib文件夹

解决：File > Project Structure > Artifacts > 在右侧Output Layout右击项目名，选择Put into Output Root。执行后，在WEB-INF在增加了lib目录，里面是项目引用的jar包，点击OK。

