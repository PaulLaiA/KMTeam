---
界: Spring Framework
门: SpringMVC
纲: 
tags: ["#SpringFramework/#SpringMVC","#MVC","#BQ"]
aliases:
  - 
date: 2021-09-08
---
 #Framework #SpringFramework #SpringMVC #MVC

-   程序先执行preHandle()方法，如果该方法的返回值为true，则程序继续向下执行处理器中的方法， 否则将不再向下执行。
-   在业务处理器（控制器Controller类）处理完请求后，会执行postHandle()方法，然后通过 DispatcherServlet向客户端返回响应。
-   DispatcherServlet处理完请求后，才会执行afterCompletion()方法。