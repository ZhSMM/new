# SpringMVC重定向

Spring MVC中做form表单功能提交时，防止用户客户端后退或者刷新时重复提交问题，需要在服务端进行重定向跳转,其中redirect是直接跳转到其他页面，有以下3种方法进行重定向。

### 流程

客户发送一个请求到服务器，服务器匹配servlet，这都和请求转发一样，servlet处理完之后调用了sendRedirect()这个方法，这个方法是response的方法，所以，当这个servlet处理完之后，看到response.sendRedirect()方法，立即向客户端返回这个响应，响应行告诉客户端你必须要再发送一个请求，去访问test.jsp，紧接着客户端受到这个请求后，立刻发出一个新的请求，去请求test.jsp,这里两个请求互不干扰，相互独立，在前面request里面setAttribute()的任何东西，在后面的request里面都获得不了。可见，在sendRedirect()里面是两个请求，两个响应。

### response.sendRedirect重定向跳转

```java
@RequestMapping(value="/testredirect",method = { RequestMethod.POST, RequestMethod.GET })  
public ModelAndView testredirect(HttpServletResponse response){  
    response.sendRedirect("/index");
    return null; 
}
```

### ViewResolver直接跳转

不带参数

```java
@RequestMapping(value="/testredirect",method = { RequestMethod.POST, RequestMethod.GET })  
public  String testredirect(HttpServletResponse response){  
    return "redirect:/index";  
} 
```

带参数

```java
@RequestMapping("/testredirect")
public String testredirect(Model model, RedirectAttributes attr) {
	attr.addAttribute("test", "51gjie");//跳转地址带上test参数
    attr.addFlashAttribute("u2", "51gjie");//跳转地址不带上u2参数
	return "redirect:/user/users";
}
```

- 使用RedirectAttributes的addAttribute方法传递参数会跟随在URL后面，如上代码即为http:/index.action?test=51gjie
- 使用addFlashAttribute不会跟随在URL后面，会把该参数值暂时保存于session，待重定向url获取该参数后从session中移除，这里的redirect必须是方法映射路径，jsp无效。你会发现redirect后的jsp页面中b只会出现一次，刷新后b再也不会出现了，这验证了上面说的，b被访问后就会从session中移除。对于重复提交可以使用此来完成.
- spring mvc设置下RequestMappingHandlerAdapter 的ignoreDefaultModelOnRedirect=true,这样可以提高效率，避免不必要的检索。

### ModelAndView重定向

不带参数

```java
@RequestMapping(value="/restredirect",method = { RequestMethod.POST, RequestMethod.GET })  
public  ModelAndView restredirect(String userName){  
    ModelAndView  model = new ModelAndView("redirect:/main/index");    
    return model;  
}
```

带参数

```java
@RequestMapping(value="/toredirect",method = { RequestMethod.POST, RequestMethod.GET })  
public  ModelAndView toredirect(String userName){  
    ModelAndView  model = new ModelAndView("/main/index");   
    model.addObject("userName", userName);  //把userName参数带入到controller的RedirectAttributes
    return model;  
}
```

### 总结

1, redirect重定向可以跳转到任意服务器，可以用在系统间的跳转。
 2, Spring MVC中redirect重定向，参数传递可以直接拼接url也可以使用RedirectAttributes来处理，由于是不同的请求，重定向传递的参数会在地址栏显示，所以传递时要对中文编码进行处理。