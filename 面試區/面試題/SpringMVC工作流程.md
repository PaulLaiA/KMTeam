---
界: Spring Framework
門: SpringMVC
綱: 
tags: ["#Spring","#SpringMVC","#MVC","#BQ"]
aliases:
  - 
date: 2021-09-08
---

-   使用者發起請求，請求到達前端控制器DispatcherServlet
-   DispatcherServlet呼叫處理器對映器HandlerMapping
-   HandlerMapping根據請求url找到找到具體的Handler產生處理器對像HandlerAdapter及攔截器返回給DispatcherServlet
-   DispatcherServlet呼叫Handler，返回ModelAndView給DispatcherServlet
-   DispatcherServlet將ModelAndView交給ViewResolver檢視解析器進行解析返回View
-   DispatcherServlet對檢視進行渲染，將數據模型填充到request域
-   向返回響應結果