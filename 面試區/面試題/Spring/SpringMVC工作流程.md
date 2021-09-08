---
界: Framework
门: Spring Framework
纲: SpringMVC
创建日期: 2021-09-08
---
 #Framework #SpringFramework #SpringMVC #MVC

-   用户发起请求，请求到达前端控制器DispatcherServlet
-   DispatcherServlet调用处理器映射器HandlerMapping
-   HandlerMapping根据请求url找到找到具体的Handler生成处理器对象HandlerAdapter及拦截器返回给DispatcherServlet
-   DispatcherServlet调用Handler，返回ModelAndView给DispatcherServlet
-   DispatcherServlet将ModelAndView交给ViewResolver视图解析器进行解析返回View
-   DispatcherServlet对视图进行渲染，将数据模型填充到request域
-   向返回响应结果