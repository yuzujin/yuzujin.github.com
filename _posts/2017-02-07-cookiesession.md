---
layout: post
title: "Cookie和Session的区别"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Http是一个无状态协议，http协议状态的保持通常有两种实现方式：cookie和session。

- Cookies：通过客户端来保持状态信息

  Cookie是服务器发给客户端的特殊信息

  Cookie是以文本的方式保存在客户端，每次请求时都带上它

- Session：通过服务器端来保持状态信息

  Session是服务器和客户端之间的一系列的交互动作

  服务器为每个客户端开辟内存空间，从而保持状态信息

  由于需要客户端也要持有一个标识(id)，因此，也要求服务器端和客户端传输该标识，

  标识(id)可以借助Cookie机制或者其他的途径来保存


**COOKIE机制**

1) Cookie的基本特点

   - Cookie保存在客户端

   - 只能保存字符串对象，不能保存对象类型

   - 需要客户端浏览器的支持：客户端可以不支持，浏览器用户可能会禁用Cookie

2) 采用Cookie需要解决的问题

   - Cookie的创建

     通常是在服务器端创建的(当然也可以通过javascript来创建)

     服务器通过在http的响应头加上特殊的指示，那么浏览器在读取这个指示后就会生成相应的cookie了

   - Cookie存放的内容

     业务信息("key","value")

     过期时间

     域和路径

   - 浏览器是如何通过Cookie和服务器通信？

     通过请求与响应，cookie在服务器和客户端之间传递

     每次请求和响应都把cookie信息加载到响应头中；依靠cookie的key传递。

**COOKIE编程**

- Cookie类

  Servlet API封装了一个类：javax.servlet.http.Cookie，封装了对Cookie的操作，包括：

  * public Cookie(String name, String value) //构造方法，用来创建一个Cookie

  * HttpServletRequest.getCookies() //从Http请求中可以获取Cookies

  * HttpServletResponse.addCookie(Cookie) //往Http响应添加Cookie

  * public int getMaxAge() //获取Cookie的过期时间值

  * public void setMaxAge(int expiry) //设置Cookie的过期时间值
	

- Cookie的创建

  Cookie是一个名值对(key=value)，而且不管是key还是value都是字符串

  Cookie visit = new Cookie("visit", "1");

- Cookie的类型——过期时间

  * 会话Cookie

    Cookie.setMaxAge(-1);//负整数

    保存在浏览器的内存中，也就是说关闭了浏览器，cookie就会丢失

  * 普通cookie

    Cookie.setMaxAge(60);//正整数，单位是秒

    表示浏览器在1分钟内不继续访问服务器，Cookie就会被过时失效并销毁(通常保存在文件中)

`注意`：

  cookie.setMaxAge(0);//等价于不支持Cookie；


**SESSION机制**


每次客户端发送请求，服务断都检查是否含有sessionId。

如果有，则根据sessionId检索出session并处理；如果没有，则创建一个session，并绑定一个不重复的sessionId。

1) 基本特点

   - 状态信息保存在服务器端。这意味着安全性更高

   - 通过类似与Hashtable的数据结构来保存

   - 能支持任何类型的对象(session中可含有多个对象)

2) 保存会话id的技术

   - Cookie

     这是默认的方式，在客户端与服务器端传递JSeesionId

     缺点：客户端可能禁用Cookie

   - 表单隐藏字段

     在被传递回客户端之前，在 form 里面加入一个hidden域，设置JSeesionId：

     <input type=hidden name=jsessionid value="3948E432F90932A549D34532EE2394" />

   - URL重写

     直接在URL后附加上session id的信息

     HttpServletResponse对象中，提供了如下的方法：

     encodeURL(url); //url为相对路径

**SESSION编程**

1) HttpSession接口

   Servlet API定义了接口：javax.servlet.http.HttpSession， Servlet容器必须实现它，用以跟踪状态。

   当浏览器与Servlet容器建立一个http会话时，容器就会通过此接口自动产生一个HttpSession对象

2) 获取Session

   HttpServletRequest对象获取session，返回HttpSession：

   request.getSession(); //表示如果session对象不存在，就创建一个新的会话

   request.getSession(true); //等价于上面这句；如果session对象不存在，就创建一个新的会话

   request.getSession(false); //表示如果session对象不存在就返回 null，不会创建新的会话对象

3) Session存取信息

   session.setAttribute(String name,Object o) //往session中保存信息

   Object session.getAttribute(String name) //从session对象中根据名字获取信息

4) 设置Session的有效时间

   public void setMaxInactiveInterval(int interval)

   设置最大非活动时间间隔，单位秒；

   如果参数interval是负值，表示永不过时。零则是不支持session。

   通过配置web.xml来设置会话超时，单位是分钟
   
```
   <seesion-config>

       <session-timeout>1</session-timeout>

   </session-config>
```

   允许两种方式并存，但前者优先级更高

5) 其他常用的API

   HttpSession.invalidate() //手工销毁Session

   boolean HttpSession.isNew() //判断Session是否新建

   如果是true，表示服务器已经创建了该session，但客户端还没有加入(还没有建立会话的握手)

   HttpSession.getId() //获取session的id
   

**两种状态跟踪机制的比较**

|Cookie|Session|
|:----:|:-------:|
|保持在客户端 | 保存在服务器端|
|只能保持字符串对象 | 支持各种类型对象|
|通过过期时间值区分Cookie的类型 | 需要sessionid来维护与客户端的通信|
|类型1：会话Cookie——负数 | 保存会话技术1：Cookie(默认)|
|类型1：普通Cookie——正数 | 保存会话技术2：表单隐藏字段|
|类型3：不支持Cookie——0  | 保存会话技术3：url重写|

**应用领域**

web交易需要保存状态的时候，都可以使用，比如在分布式场景下可以利用分布式session技术等。