# SpringMVC参数保存

## Spring参数保存方式

```
package top.songfang.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.SessionAttributes;
import org.springframework.web.servlet.ModelAndView;
import top.songfang.entity.Student;

import javax.servlet.http.HttpSession;
import java.util.Map;

/**
 *  处理模型数据：如果跳转时需要带数据：V、M，则可以使用以下方式：
 *    数据放在request作用域
 *    ModeAndView、ModelMap、Map、Model、@SessionAttributes、@ModelAttribute
 */

@SessionAttributes({"student0","hi"})
@Controller
public class TestController {

    // 下列方法全是把参数加到request域里面，希望同时放入Session里面时，在类前面加
    // 默认情况下Spring MVC将模型中的数据存储到request域中。当一个请求结束后，数据就失效了。
    //     如果要跨页面使用。那么需要使用到session。而@SessionAttributes注解就可以使得模型中
    //     的数据存储一份到session域中。
    //     @SessionAttributes注解只能在类上使用，不能在方法上使用
    // @SessionAttributes(types=Student.class)
    // @SessionAttributes(value="student,student0,...")

    // 报错：org.springframework.web.HttpSessionRequiredException
    // 当使用SessionAttributes注解缓存对象时，handler处理类的处理方法中存在对应的入参时，
    // SpringMVC在implicitModel中找不到对应的对象时，SpringMVC会去判断是否使用SessionAttributes
    // 注解缓存了对应的类。如果缓存了，就会去查找对应的对象，如果找不到，就会抛出此异常。


    // ModelAndView:既有数据，又有视图 Model -M和View -V
    @RequestMapping(value = "test0.action")
    public ModelAndView test0(Student student){
        // 设置view:views/success.jsp
        ModelAndView mv=new ModelAndView("student");

        // 添加参数，K-V对，可以添加任意类型
        // 相当于request.setAttribute("Student",student);
        mv.addObject("student0",student);
        return mv;
    }

    @RequestMapping(value = "test1.action")
    public String test1(Student student,ModelMap modelMap){
        modelMap.put("student0",student);
        return "student";
    }

    @RequestMapping(value = "test2.action")
    public String test2(Student student,Model model){
        model.addAttribute("student0",student);
        return "student";
    }

    @RequestMapping(value = "test3.action")
    public String test3(Student student, Map<String,Object> map, HttpSession httpSession){
        map.put("student0",student);
        System.out.println(httpSession.getAttribute("student"));
        return "student";
    }

    @RequestMapping(value = "test4.action")
    public String test4(Map<String,Object> map){
        map.put("hi","hvg");
        return "student";
    }
}

```

