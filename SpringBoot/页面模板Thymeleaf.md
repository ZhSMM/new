# 页面模板Thymeleaf

## 引入依赖

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

## 配置Thymeleaf

```java
## thymeleaf 缓存是否开启，开发时建议关闭，否则更改页面后不会实时展示效果
spring.thymeleaf.cache=false
## thymeleaf 编码格式
spring.thymeleaf.encoding=UTF-8
## thymeleaf 对HTML的校验很严格，用这个去除thymeleaf严格校验
spring.thymeleaf.mode=LEGACYHTML5    
## thymeleaf 模板文件前缀
spring.thymeleaf.prefix=classpath:/templates/
## thymeleaf 模板文件后缀
spring.thymeleaf.suffix=.html
```

## 测试

创建一个Controller和HTML进行测试。

```java
@Controller
public class IndexController {
    @GetMapping("/")
    public String index(ModelMap modelMap){
        modelMap.addAttribute("msg","Hello,hh");
        return "index";
    }
}

// 在src/main/resources/templates下新建index.html(需要结合上面的配置文件)
// 使用th:text="${msg}"来接受后台的数据。
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 th:text="${msg}"></h1>
</body>
</html>
    
// 访问http://localhost:8080既可以看到输出
```

## Thymeleaf的常用语法

- th:text 设置当前元素的文本内容
- th:value 设置当前元素的值
- th:each 循环遍历元素
- th:attr 设置当前元素的属性
- th:if th:switch th:case th:unless 用作条件判断
- th:insert th:replace th:include 代码块引入，一般用作提取公共文件或者引用公共静态文件

内置方法：

- #numbers 数字方法
- #dates 日期方法
- #calendars 日历方法
- #strings 字符串方法
- #lists 集合方法
- #maps 对象方法

## 文件上传下载

1. 在src/main/resources/templates新建index.html文件

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title th:text="${msg}">Title</title>
   </head>
   <body>
   <p>单文件上传</p>
   <form action="upload" method="post" enctype="multipart/form-data">
       文件：<input type="file" name="file">
       <input type="submit">
   </form>
   <hr/>
   <p>文件下载</p>
   <a href="download">下载文件</a>
   <hr/>
   <p>多文件上传</p>
   <form action="batch" method="post" enctype="multipart/form-data">
       <p>文件1：<input type="file" name="file"></p>
       <p>文件2：<input type="file" name="file"></p>
       <p><input type="submit" value="上传"></p>
   </form>
   </body>
   </html>
   ```

2. 创建IndexController，进行url映射，需要进行配置

   ```java
   import org.springframework.stereotype.Controller;
   import org.springframework.ui.ModelMap;
   import org.springframework.web.bind.annotation.GetMapping;
   
   @Controller
   public class IndexController {
       @GetMapping("/")
       public String index(ModelMap modelMap){
           modelMap.addAttribute("msg","文件上传下载");
           return "index";
       }
   }
   ```

3. 创建FileController用于文件和下载测试：

   1. 单个上传方法：根据页面上使用的input标签的name值获取对应内容，因为是文件，所以可以直接使用MultipartFile对象来接受文件，由于只是简单测试，所以用File类自带的transferTo方法直接将文件存入对应存储位置；
   2. 批量上传方法：获取页面内容的方式与单个文件方法一致，不同的是取得文件后，用BufferedOutputStream流来进行上传；
   3. 下载方法：例为对固定位置的文件进行下载。

   ```java
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   import org.springframework.web.bind.annotation.*;
   import org.springframework.web.multipart.MultipartFile;
   import org.springframework.web.multipart.MultipartHttpServletRequest;
   
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   import java.io.*;
   import java.util.List;
   
   @RestController
   public class FileController {
       private static final String filePath="D:/";
       private static final Logger log= LoggerFactory.getLogger(FileController.class);
   
       @RequestMapping(value = "/upload")
       public String upload(@RequestParam("file")MultipartFile file){
           try{
               if(file.isEmpty()){
                   return "文件为空";
               }
               // 获取文件夹
               String fileName=file.getOriginalFilename();
               log.info("上传的文件名为："+fileName);
               // 设置文件存储路径
               String path=filePath+fileName;
               File dest=new File(path);
               // 检测是否存在目录
               if (!dest.getParentFile().exists()){
                   dest.getParentFile().mkdirs(); //新建文件夹
               }
               file.transferTo(dest);
               return "上传成功";
           }catch(IllegalStateException e){
               e.printStackTrace();;
           }catch (IOException e){
               e.printStackTrace();
           }
           return "上传失败";
       }
   
       @PostMapping("/batch")
       public String handleFileUpload(HttpServletRequest request){
           List<MultipartFile> files = ((MultipartHttpServletRequest)request).getFiles("file");
           MultipartFile file=null;
           BufferedOutputStream stream=null;
           for (int i = 0; i < files.size(); i++) {
               file=files.get(i);
               if (!file.isEmpty()){
                   try{
                       byte[] bytes=file.getBytes();
                       stream=new BufferedOutputStream(new FileOutputStream(
                               new File(filePath+file.getOriginalFilename())
                       ));
                       stream.write(bytes);
                       stream.close();
                   }catch(Exception e){
                       stream=null;
                       return "第"+i+"个文件上传失败===》"+e.getMessage();
                   }
               }else {
                   return "第"+i+"个文件上传失败，因为文件为空";
               }
           }
           return "上传成功";
       }
   
       @GetMapping("/download")
       public String downloadFile(HttpServletResponse response){
           String fileName="页面模板Thymeleaf.md";
           if(fileName!=null){
               File file=new File(filePath+fileName);
               if(file.exists()){
                   response.setHeader("Content-Disposition","attachment;fileName="+fileName);
                   byte[] buffer=new byte[1024];
                   FileInputStream fis=null;
                   BufferedInputStream bis=null;
                   try{
                       fis=new FileInputStream(file);
                       bis=new BufferedInputStream(fis);
                       OutputStream os=response.getOutputStream();
                       int i=bis.read(buffer);
                       while(i!=-1){
                           os.write(buffer,0,i);
                           i=bis.read(buffer);
                       }
                       return "下载成功";
                   }catch (Exception e){
                       e.printStackTrace();
                   }finally {
                       if(bis!=null){
                           try{
                               bis.close();
                           }
                           catch (IOException e){
                               e.printStackTrace();
                           }
                       }
                       if(fis!=null){
                           try{
                               fis.close();
                           }
                           catch (IOException e){
                               e.printStackTrace();
                           }
                       }
                   }
               }
           }
           return "下载失败";
       }
   }
   
   ```

   

