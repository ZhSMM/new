# Rest风格

## 概述

REST：Representational State Transfer

REST风格：就是用URL定位资源，用HTTP动词（GET、POST、DELETE、PUT）描述操作。

在设计web接口的时候，REST主要用于定义接口名，接口名一般是用名词写，不用动词，然后操作通过请求类型来区分，返回值一般为JSON类型。比如：friend的增删改查：

- 增：url:generalcode.cn/v/friends 接口类型：POST
- 删：url:generalcode.cn/v/friends 接口类型：DELETE
- 改：url:generalcode.cn/v/friends 接口类型：PUT
- 查：url:generalcode.cn/v/friends 接口类型：GET

反例：url:generalcode.cn/v/getfriends就不符合要求。

## 优点

那这种风格的接口有什么好处呢？前后端分离。前端拿到数据只负责展示和渲染，不对数据做任何处理。后端处理数据并以JSON格式传输出去，定义这样一套统一的接口，在web，ios，android三端都可以用相同的接口。



