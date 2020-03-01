# @ModelAttribute

通过@ModelAttribute修饰的方法，会在每次请求==**该类的任意方法时都会先执行**==（设计思想：一个控制器只做一件事），并且该方法的参数map.put()可以将对象放入即将查询的参数中。

约定：map.put(k,v) 其中的k必须是即将查询的方法参数的首字母小写。

​			或者用在方法参数之前用@ModelAttribute("key")声明。

```java
@Controller
public class TestController {
    // 在方法执行之前执行
    @ModelAttribute
    public void queryStudentById(Map<String,Object> map){
        Student student=new Student(12,"zhajn",32);
        // 约定：map的key就是方法参数类型的首字母小写一样，就能自动传值
        // 如果不一致，可以在方法参数加@ModelAttribute("参数名")
        map.put("student",student);
    }
    @RequestMapping("test0.action")
    public String testModelAttribute(
        @ModelAttribute("student") Student student){
        return "output";
    }
}
```

