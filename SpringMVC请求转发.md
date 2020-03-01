# SpringMVC请求转发

SpringMVC支持的转发方式有两种：forward和redirect

## forward

用forward转发方式url不会改变；

## redirect

```java
// 在使用redirect进行重定向时请求的URL连接会发生变化，当转发时需要参数时，可以采用如下办法
// 返回String类型，将参数拼到里面：
	@RequestMapping(value = "test", method = RequestMethod.GET)   
    public RedirectView postPayAmount(HttpSession session,
                                      @RequestParam("hh")String str) {   
        return "redirect:/tolink?hh=str";
    }
// 常规做法，重定向之前把参数放进Session中，在重定向之后的controller中把参数从
// Session中取出并放进ModelAndView

// 使用RedirectAttributes类
// 使用时：需要在需要重定向的controller的方法参数中添加RedirectAttributes类型的参数，
//        然后把需要重定向之后也能够获取的参数放进去即可
	@RequestMapping("/user/index.html")
	public ModelAndView userIndex() {
		return new ModelAndView("user/index");
	}

	@RequestMapping("/testRedirect.html")
	public ModelAndView testRedirect(@RequestParam("username") String username,RedirectAttributes redirectAttributes){
		ModelAndView mAndView = new ModelAndView("redirect:/user/index.html");
		
		User user = new User();
		user.setName(username);
		//URL后面拼接参数
		redirectAttributes.addFlashAttribute("user", user);
		
		return mAndView;
	}





    @RequestMapping(value = "postPayAmount", method = RequestMethod.GET)   
    public RedirectView postPayAmount(HttpSession session,ModelMap map) {   
        return new RedirectView(WsUrlConf.URI_PAY,true,false,false);//最后的参数为false代表以post方式提交请求   
    }  
return new ModelAndView(new RedirectView("xxx.do"), map);  
```

