---
layout: post
title: "SpringMVC框架理解"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Spring Web MVC是一种基于Java的实现了Web MVC设计模式的请求驱动类型的轻量级Web框架，即使用了MVC架构模式的思想，将web层进行职责解耦，基于请求驱动指的就是使用请求-响应模型，框架的目的就是帮助我们简化开发。

Spring的MVC框架主要由前端控制器(DispatcherServlet)、(处理器映射)HandlerMapping、控制器(Handler或Controller)、处理器适配器(HandlerAdapter或Service)、(视图解析器)View Resolver、(视图)View组成。

![image](https://github.com/yuzujin/yuzujin.github.com/blob/master/images/spring3.jpg?raw=true)

运行流程：

1. 客户端请求提交到DispatcherServlet

2. 由DispatcherServlet控制器查询一个或多个HandlerMapping，找到处理请求的Controller

3. DispatcherServlet将请求提交到Controller

4. Controller调用业务逻辑处理后，返回ModelAndView

5. DispatcherServlet查询一个或多个ViewResoler视图解析器，找到ModelAndView指定的视图

6. 视图负责将结果显示到客户端


## SpringMVC组件

**DispatcherServlet**

DispatcherServlet是整个Spring MVC的核心。所有的请求都有经过它来统一分发。在DispatcherServlet将请求分发给Spring Controller之前，需要借助于Spring提供的HandlerMapping定位到具体的Controller。其主要工作有以下三项：

1. 截获符合特定格式的URL请求。
2. 初始化DispatcherServlet上下文对应的WebApplicationContext，并将其与业务层、持久化层的WebApplicationContext建立关联。
3. 初始化Spring MVC的各个组成组件，并装配到DispatcherServlet中。

**HandlerMapping**

能够完成客户请求到Controller映射。

**Controller**

需要为并发用户处理上述请求，因此实现Controller接口时，必须保证线程安全并且可重用。

Controller将处理用户请求，这和Struts Action扮演的角色是一致的。一旦Controller处理完用户请求，则返回ModelAndView对象给DispatcherServlet前端控制器，ModelAndView中包含了模型（Model）和视图（View）。

从宏观角度考虑，DispatcherServlet是整个Web应用的控制器；从微观考虑，Controller是单个Http请求处理过程中的控制器，而ModelAndView是Http请求过程中返回的模型（Model）和视图（View）。

**ViewResolver**

Spring提供的视图解析器（ViewResolver）在Web应用中查找View对象，从而将相应结果渲染给客户。

## SpringMVC注解

- @Controller

  负责注册一个bean 到spring 上下文中
  
- @Service
   
  服务层组件，用于标注业务层组件,表示定义一个bean，自动根据bean的类名实例化一个首写字母为小写的bean

- @RequestMapping

　注解为控制器指定可以处理哪些 URL 请求

- @RequestBody

　该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上 ,再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上

- @ResponseBody

　该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区

- @ModelAttribute 　　　

　在方法定义上使用 @ModelAttribute 注解：Spring MVC 在调用目标处理方法前，会先逐个调用在方法级上标注了@ModelAttribute 的方法

　在方法的入参前使用 @ModelAttribute 注解：可以从隐含对象中获取隐含的模型数据中获取对象，再将请求参数 –绑定到对象中，再传入入参将方法入参对象添加到模型中 

- @RequestParam　

　在处理方法入参处使用 @RequestParam 可以把请求参 数传递给请求方法

- @PathVariable

　绑定 URL 占位符到入参

- @ExceptionHandler

　注解到方法上，出现异常时会执行该方法

- @ControllerAdvice

　使一个Contoller成为全局的异常处理类，类中用@ExceptionHandler方法注解的方法可以处理所有Controller发生的异常